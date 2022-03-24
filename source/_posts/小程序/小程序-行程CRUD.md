---
title: 小程序-行程CRUD
tags: 想要车小程序
categories: 
- 学习记录
- 想要车小程序开发
- 后端
cover: https://images.pexels.com/photos/3112898/pexels-photo-3112898.jpeg?auto=compress&cs=tinysrgb&dpr=2&w=500
date: 2022-03-12 18:18:27
updated: 2022-03-12 18:18:30
keywords: 行程CRUD
---
# 结构定义

在`./rental/rental.proto`定义业务，定义Trip

**value object VS entity**

1. value object：值相等就是同一个
2. entity：特有属性相等，其他不等，才是同一个，比如ID

```protobuf
syntax = "proto3";
package rental.v1;
option go_package="coolcar/rental/api/gen/v1;rentalpb";
// value object
message Location{
    double latitude = 1;
    double longitude = 2;
}
// 移动距离和钱是实时的，绑定的
message LocationStatus{
    Location location = 1;
    int32 fee_cent = 2;
    double km_driven = 3;
    string poi_name = 4;
}
// value object
enum TripStatus {
    TS_NOT_SPECIFIED=0
    IN_PROGRES = 1;
    FINISHED = 2;
   
}
// 定义trip entity
message TripEntity{
    string id = 1;
    trip trip = 2;
}
// value object
message Trip{
	string accountID = 1;
    string carID = 2;
    LocationStatus start = 3;
    LocationStatus current = 4;
    LocationStatus end = 5;
    TripStatus status = 6;
}

message CreateTripRequest{
    string start = 1;
    string car_id = 2;
}
message GetTripRequest{
    string id = 1;
}

// rpc GetTrips(GetTripsRequest) returns (GetTripsResponse)
message GetTripsRequest{
	TripStatus status = 1;
}
message GetTripsResponse{
	repeated TripEntity trips = 1;
}
// 只能更新地点其他的都不能改
message UpdateTripRequest{
	string id = 1;
	Location current = 2;
	bool end_trip = 3;
}
service TripService{
    rpc CreateTrip (CreateTripRequest) returns (TripEntity);
    rpc GetTrip(GetTripRequest) returns (Trip) 
    rpc GetTrips(GetTripsRequest) returns (GetTripsResponse)
    rpc UpdateTrip(UpdateTripRequest) returns (Trip)
}
```

假设将`TripEntity`的id放在`Trip`中，那么这个id是自己生成的还是前端给的，这个trip在创建的时候，要不要填写id。如果想表达系统会生成，不用填写id，但是在代码无法说明，除非加了注释。

生成代码

```bash
set PROTO_PATH=.\rental\api
set GO_OUT_PATH=.\rental\api\gen\v1
mkdir %GO_OUT_PATH%
protoc -I=%PROTO_PATH% --go_out=plugins=grpc,paths=source_relative:%GO_OUT_PATH% rental.proto
protoc -I=%PROTO_PATH% --grpc-gateway_out=paths=source_relative,grpc_api_configuration=%PROTO_PATH%/rental.yaml:%GO_OUT_PATH% rental.proto

set PBTS_BIN_DIR= ..\wx\miniprogram\node_modules\.bin
set PBTS_OUT_DIR= ..\wx\miniprogram\service\proto_gen\rental
mkdir %PBTS_OUT_DIR%

%PBTS_BIN_DIR%\pbjs -t static -w es6 ./rental/api/rental.proto --no--create --no--decode --no--verify --no--delimited -o %PBTS_OUT_DIR%/rental_pb.js

%PBTS_BIN_DIR%\pbts -o %PBTS_OUT_DIR%/rental_pb.d.ts %PBTS_OUT_DIR%/rental_pb.js
```

# 行程创建

## `/rental/trip/dao/monogo.go`

```go
type Mongo struct {
	col *mongo.Collection
}

func NewMongo(db *mongo.Database) *Mongo {
	return &Mongo{
		col: db.Collection("trip"),
	}
}

type TripRecord struct {
	mgo.IDField      `bson:"inline"`
	mgo.UpdatedField `bson:"inline"`
	Trip             *rentalpb.Trip `bson:"trip"`
}
// TODO: 同一个account最多只能有一个进行中的trip
// TODO: 强类型化tripID
// TODO: 表格驱动测试
func (m *Mongo) CreateTrip(c context.Context, trip *rentalpb.Trip) (*TripRecord, error) {
	r := &TripRecord{
		Trip: trip,
	}
	// 这两个参数是不固定的，交给mgo管理，在测试的时候可以固定
	r.ID = mgo.NewObjID()
	r.UpdatedAt = mgo.UpdateAt()
	_, err := m.col.InsertOne(c, r)
	if err != nil {
		return nil, err
	}
	return r, nil
}
```

## 测试代码：

```go
func TestCreateTrip(t *testing.T) {
	c := context.Background()
	s := "mongodb://localhost:27017"
	mc, err := mongo.Connect(c, options.Client().ApplyURI(s))
	if err != nil {
		t.Fatalf("cannot connection db: %v", err)
	}
	m := NewMongo(mc.Database("coolcar"))
	trip, err := m.CreateTrip(c, &rentalpb.Trip{
		AccountID: "account1",
		CarID:     "car1",
		Start: &rentalpb.LocationStatus{
			PoiName: "startpoint",
			Location: &rentalpb.Location{
				Latitude:  30,
				Longitude: 120,
			},
		},
		End: &rentalpb.LocationStatus{
			PoiName:  "endpoint",
			FeeCent:  10000,
			KmDriven: 100000,
			Location: &rentalpb.Location{
				Latitude:  35,
				Longitude: 125,
			},
		},
		Current: &rentalpb.LocationStatus{
			PoiName:  "currentpoint",
			FeeCent:  10000,
			KmDriven: 100000,
			Location: &rentalpb.Location{
				Latitude:  35,
				Longitude: 125,
			},
		},
		Status: rentalpb.TripStatus_FINISHED,
	})
	if err != nil {
		t.Errorf("cannot create trip: %v", err)
	}
	// +v 用key：value显示
	t.Errorf("%+v", s)
	t.Errorf("insert row %s with updateat %v", trip.ID, trip.UpdatedAt)
}
func TestMain(m *testing.M) {
	os.Exit(mongotesting.RunWithMongoInDocker(m, &mongoURI))
}
```

# 行程查询

```go
func (m *Mongo) GetTrip(c context.Context, id string, accountID auth.AccountID) (*TripRecord, error) {
	objID, err := primitive.ObjectIDFromHex(id)
	if err != nil {
		return nil, fmt.Errorf("invalid id: %v", err)
	}
	res := m.col.FindOne(c, bson.M{
		mgo.IDFieldName: objID,
		accountIdField:  accountID,
	})
	if err := res.Err(); err != nil {
		return nil, err
	}
	var tr TripRecord
	err = res.Decode(&tr)
	if err != nil {
		return nil, fmt.Errorf("cannot decode : %v", err)
	}
	return &tr, err
}
```

# 强类型化tripID

`/shared/id/id.go`让字段一看就知道意义，有助于逻辑开发

```go
package id

type AccountID string
// fmt.Stringer类型，是实现了String()函数的类型。
func (a AccountID) String() string {
	return string(a)
}

type TripID string

func (t TripID) String() string {
	return string(t)
}
```

`/shared/mongo/objid/objid.go`

```go
package objid

import (
	"coolcar/shared/id"
	"fmt"

	"go.mongodb.org/mongo-driver/bson/primitive"
)
// 任何实现了String()函数的字段
func FromID(id fmt.Stringer) (primitive.ObjectID, error) {
	return primitive.ObjectIDFromHex(id.String())
}
func MustFromID(id fmt.Stringer) primitive.ObjectID {
	oid, err := FromID(id)
	if err != nil {
		panic(err)
	}
	return oid
}
func ToAccountID(oid primitive.ObjectID) id.AccountID {
	return id.AccountID(oid.Hex())
}

func ToTripID(oid primitive.ObjectID) id.TripID {
	return id.TripID(oid.Hex())
}
```

之后将错误的地方修改为`id.accountID`

# 同一个account最多只能有一个进行中的trip

只有一条`status:1`的数据，在mongodb中有下面机制控制这个操作。

如果设置了唯一索引，新插入文档时，要求 key 的值是唯一的，不能有重复的出现

```json
db.account.createIndex({
	open_id: 1,
},{
	unique: true,
})
```

```json
db.trip.createIndex({
    "trip.accountid":1,//这个1表示从小到大的意思
    "trip.status":1,
},{
    unique: true,//以上面的字段不能重复基础创建索引
    partialFilterExpression:{
        "trip.status":1,//trip.status为1的情况下只能有一个
    }
})
```

上面的操作，在go语言的代码如下：

```go
func SetupIndex(c context.Context, d *mongo.Database) error {
	_, err := d.Collection("account").Indexes().CreateOne(c, mongo.IndexModel{
		Keys: bson.D{
			{Key: "open_id", Value: 1},
		},
		Options: options.Index().SetUnique(true),
	})
	if err != nil {
		return err
	}
	_, err = d.Collection("trip").Indexes().CreateOne(c, mongo.IndexModel{
		Keys: bson.D{
			{Key: "trip.accountid", Value: 1},
			{Key: "trip.status", Value: 1},
		},
		Options: options.Index().SetUnique(true).SetPartialFilterExpression(bson.M{
			"trip.status": 1,
		}),
	})
	return err
}
```



# 测试

测试的时候比较测试值和真实值

```
go get -u github.com/google/go-cmp/cmp
```

```go
if diff := cmp.Diff(trip,got,protocmp.Transform()); diff!=""{
		t.Errorf("result differs;-want +got: %s",diff)
	}
```

![image-20220312165611412](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203121656463.png)

```go
func TestCreateTrip(t *testing.T) {
	c := context.Background()
	mc, err := mongotesting.NewClient(c)
	if err != nil {
		t.Fatalf("cannot connection db: %v", err)
	}
	db := mc.Database("coolcar")
	err = mongotesting.SetupIndex(c, db)
	if err != nil {
		t.Fatalf("cannot SetupIndex : %v", err)
	}
	m := NewMongo(db)

	cases := []struct {
		name       string
		tripID     string
		accountID  string
		tripStatus rentalpb.TripStatus
		wantErr    bool
	}{
		{
			name:       "finised",
			tripID:     "622c60186800fc9e2ca1480f",
			accountID:  "account1",
			tripStatus: rentalpb.TripStatus_FINISHED,
		},
		{
			name:       "another_finised",
			tripID:     "622c60186800fc9e2ca1480d",
			accountID:  "account1",
			tripStatus: rentalpb.TripStatus_FINISHED,
		},
		{
			name:       "in_progress",
			tripID:     "622c60186800fc9e2ca1480c",
			accountID:  "account1",
			tripStatus: rentalpb.TripStatus_IN_PROGRES,
		},
		{
			name:       "another_in_progress",
			tripID:     "622c60186800fc9e2ca1480e",
			accountID:  "account1",
			tripStatus: rentalpb.TripStatus_IN_PROGRES,
			wantErr:    true,
		},
		{
			name:       "in_progress_by_another_account",
			tripID:     "622c60186800fc9e2ca1481e",
			accountID:  "account2",
			tripStatus: rentalpb.TripStatus_IN_PROGRES,
		},
	}

	for _, cc := range cases {
		mgo.NewObjID = func() primitive.ObjectID {
			return objid.MustFromID(id.TripID(cc.tripID))
		}
		tr, err := m.CreateTrip(c, &rentalpb.Trip{
			AccountID: cc.accountID,
			Status:    cc.tripStatus,
		})
		if cc.wantErr {
			if err == nil {
				t.Errorf("%s:error expected; got none", cc.name)
			}
			continue
		}
		if err != nil {
			t.Errorf("%s:error creating trip: %v", cc.name, err)
			continue
		}

		if tr.ID.Hex() != cc.tripID {
			t.Errorf("%s: incorrect trip id;want: %q;got:%q", cc.name, cc.tripID, tr.ID.Hex())
		}
	}

}
func TestGetTrip(t *testing.T) {
	c := context.Background()
	mc, err := mongotesting.NewDefalutClient(c)
	if err != nil {
		t.Fatalf("cannot connection db: %v", err)
	}
	m := NewMongo(mc.Database("coolcar"))
	acct := "account1"
	// 如果运行整个package的时候，上面的TestCreateTrip，以及固定住了ObjID,，所以要在这里进行重置
	mgo.NewObjID = primitive.NewObjectID
	trip, err := m.CreateTrip(c, &rentalpb.Trip{
		AccountID: acct,
		CarID:     "car1",
		Start: &rentalpb.LocationStatus{
			PoiName: "startpoint",
			Location: &rentalpb.Location{
				Latitude:  30,
				Longitude: 120,
			},
		},
		End: &rentalpb.LocationStatus{
			PoiName:  "endpoint",
			FeeCent:  10000,
			KmDriven: 100000,
			Location: &rentalpb.Location{
				Latitude:  35,
				Longitude: 125,
			},
		},
		Current: &rentalpb.LocationStatus{
			PoiName:  "currentpoint",
			FeeCent:  10000,
			KmDriven: 100000,
			Location: &rentalpb.Location{
				Latitude:  35,
				Longitude: 125,
			},
		},
		Status: rentalpb.TripStatus_FINISHED,
	})
	if err != nil {
		t.Errorf("cannot create trip: %v", err)
	}
	// +v 用key：value显示
	// t.Errorf("insert row %s with updateat %v", trip.ID, trip.UpdatedAt)
	got, err := m.GetTrip(c, objid.ToTripID(trip.ID), id.AccountID(acct))
	// got.Trip.Current.PoiName = "badPoiName" 测试 cmp.Diff
	if err != nil {
		t.Errorf("cannot get trip : %v", err)
	}
	if diff := cmp.Diff(trip, got, protocmp.Transform()); diff != "" {
		t.Errorf("result differs;-want +got: %s", diff)
	}
}
func TestMain(m *testing.M) {
	os.Exit(mongotesting.RunWithMongoInDocker(m))
}
```

## 查询多个行程

```go
// GetTrips gets trips for the account by status.
// If status is not specified, gets all trips for the account.
func (m *Mongo) GetTrips(c context.Context, accountID id.AccountID, status rentalpb.TripStatus) ([]*TripRecord, error) {
	filter := bson.M{
		accountIdField: accountID.String(),
	}
	if status != rentalpb.TripStatus_TS_NOT_SPECIFIED {
		filter[statusField] = status
	}

	res, err := m.col.Find(c, filter)
	if err != nil {
		return nil, err
	}
	var trips []*TripRecord
	for res.Next(c) {
		var trip TripRecord
		err := res.Decode(&trip)
		// 其中一行出错了的处理方式
		if err != nil {
			return nil, err
		}
		trips = append(trips, &trip)
	}
	return trips, nil
}
```

### 测试

```go
func TestGetTrips(t *testing.T) {
	rows := []struct {
		id        string
		accountID string
		status    rentalpb.TripStatus
	}{
		{
			id:        "632c60186800fc9e2ca1480d",
			accountID: "account_id_for_get_trips",
			status:    rentalpb.TripStatus_FINISHED,
		},
		{
			id:        "642c60186800fc9e2ca1480d",
			accountID: "account_id_for_get_trips",
			status:    rentalpb.TripStatus_FINISHED,
		},
		{
			id:        "652c60186800fc9e2ca1480d",
			accountID: "account_id_for_get_trips",
			status:    rentalpb.TripStatus_FINISHED,
		},
		{
			id:        "682c60186800fc9e2ca1480d",
			accountID: "account_id_for_get_trips",
			status:    rentalpb.TripStatus_FINISHED,
		},
		{
			id:        "662c60186800fc9e2ca1480d",
			accountID: "account_id_for_get_trips",
			status:    rentalpb.TripStatus_IN_PROGRES,
		},
		{
			id:        "672c60186800fc9e2ca1480d",
			accountID: "account_id_for_get_trips_1",
			status:    rentalpb.TripStatus_IN_PROGRES,
		},
	}
	c := context.Background()
	mc, err := mongotesting.NewClient(c)
	if err != nil {
		t.Fatalf("cannot connect mongodb : %v", err)
	}
	m := NewMongo(mc.Database("coolcar"))
	for _, r := range rows {
		mgo.NewObjIDWithValue(id.TripID(r.id))
		_, err := m.CreateTrip(c, &rentalpb.Trip{
			AccountID: r.accountID,
			Status:    r.status,
		})
		if err != nil {
			t.Fatalf("cannot create rows : %v", err)
		}

	}

	cases := []struct {
		name       string
		accountID  string
		status     rentalpb.TripStatus
		wantCount  int
		wantOnlyID string
	}{
		{
			name:      "get_all",
			accountID: "account_id_for_get_trips",
			status:    rentalpb.TripStatus_TS_NOT_SPECIFIED,
			wantCount: 5,
		},
		{
			name:      "get_in_progress",
			accountID: "account_id_for_get_trips",
			status:    rentalpb.TripStatus_IN_PROGRES,
			wantCount: 1,
		},
	}

	for _, cc := range cases {
		t.Run(cc.name, func(t *testing.T) {
			tr, err := m.GetTrips(context.Background(), id.AccountID(cc.accountID), cc.status)
			if err != nil {
				t.Errorf("cannot get trips: %v", err)
			}
			if cc.wantCount != len(tr) {
				t.Errorf("incorrect result count:want:%d,got:%d", cc.wantCount, len(tr))
			}
			if cc.wantOnlyID != "" && len(tr) > 0 {
				if cc.wantOnlyID != tr[0].ID.Hex() {
					t.Errorf("only_id incorrect;want:%q,got:%q", cc.wantOnlyID, tr[0].ID.Hex())
				}
			}
		})
	}
}
```

# 行程跟新

## 乐观锁

使用updatedAt字段来表示锁，update只有和mongo中相同才可以更新。实际代码中，需要update这个字段去查找数据

```go
func (m *Mongo) UpdateTrip(c context.Context, tripid id.TripID, accountID id.AccountID, updatedAt int64, trip *rentalpb.Trip) error {
	objID, err := objid.FromID(tripid)
	if err != nil {
		return fmt.Errorf("invalid id : %v", err)
	}
	newUpdatedAt := mgo.UpdateAt()
	res, err := m.col.UpdateOne(c, bson.M{
		mgo.IDFieldName:        objID,
		accountIdField:         accountID.String(),
		mgo.UpdatedAtFieldName: updatedAt,
	}, mgo.Set(bson.M{
		tripField:              trip,
		mgo.UpdatedAtFieldName: newUpdatedAt,
	}))
	if err != nil {
		return err
	}
	if res.MatchedCount == 0 {
		return mongo.ErrNoDocuments
	}
	return nil
}
```

## 测试

```go
func TestUpdateTrip(t *testing.T) {
	c := context.Background()
	mc, err := mongotesting.NewClient(c)
	if err != nil {
		t.Fatalf("cannot connect mongodb:%v", err)
	}
	m := NewMongo(mc.Database("coolcar"))
	tid := id.TripID("662c60186800fc9e2ca1480d")
	var now int64 = 10000
	mgo.NewObjIDWithValue(tid)
	mgo.UpdateAt = func() int64 {
		return now
	}
	aid := id.AccountID("account_for_update")
	tr, err := m.CreateTrip(c, &rentalpb.Trip{
		AccountID: aid.String(),
		Status:    rentalpb.TripStatus_IN_PROGRES,
		Start: &rentalpb.LocationStatus{
			PoiName: "start_poi",
		},
	})
	if err != nil {
		t.Fatalf("cannot create trip:%v", err)
	}
	if tr.UpdatedAt != 10000 {
		t.Fatalf("wrong updateat;want:10000,got:%d", tr.UpdatedAt)
	}
	update := &rentalpb.Trip{
		AccountID: aid.String(),
		Status:    rentalpb.TripStatus_IN_PROGRES,
		Start: &rentalpb.LocationStatus{
			PoiName: "start_poi_updated",
		},
	}
	cases := []struct {
		name           string
		now            int64
		withUppdatedAt int64
		wantErr        bool
	}{
		{
			name:           "normal_update",
			now:            20000,
			withUppdatedAt: 10000,
		},
		{
			name:           "update_with_stale_timestamp",
			now:            30000,
			withUppdatedAt: 10000,
			wantErr:        true,
		},
		{
			name:           "update_with_refetch",
			now:            40000,
			withUppdatedAt: 20000,
		},
	}

	for _, cc := range cases {
		// 用case的now替代mgo.updatedAt
		now = cc.now
		err := m.UpdateTrip(c, tid, aid, cc.withUppdatedAt, update)
		if cc.wantErr {
			if err == nil {
				t.Errorf("%s:want error: got none", cc.name)
			} else {
				continue
			}
		}
		updatedTrip, err := m.GetTrip(c, tid, aid)
		if err != nil {
			t.Errorf("%s:cannot get trip after update:%v", cc.name, err)
		}
		if cc.now != updatedTrip.UpdatedAt {
			t.Errorf("%s:incorrect updatedat:want %d,got %d", cc.name, cc.now, updatedTrip.UpdatedAt)
		}

	}
}
```

