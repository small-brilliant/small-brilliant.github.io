---
title: 身份信息的CRUD
tags: 想要车小程序
categories: 
- 学习记录
- 想要车小程序开发
- 后端
cover: https://images.pexels.com/photos/3112898/pexels-photo-3112898.jpeg?auto=compress&cs=tinysrgb&dpr=2&w=500
date: 2022-03-16 13:18:44
updated: 2022-03-16 13:18:47
keywords: 身份信息的CRUD
---
[toc]
# 服务定义

![image-20220318124339492](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203181243646.png)

`profile`可以不用看作一个实体类型，一个看成值类型

因为profile是和accountID绑定的，初始获取Profile应该是空值，如果是实体那就是nil，会报错。

下面的是加在`rental.proto`里，可以考虑重新用一个文件才存储

```protobuf
// Profile Service

enum Gender{
    G_NOT_SPECIFIED = 0;
    MALE=1;
    FEMALE=2;
}
enum IdentityStatus {
    UNSUBMITTED=0;
    PENDING =1;
    VERIFIED=2;
}
message Profile{
    Identity identity = 1;
    IdentityStatus identity_status=2;
}

message Identity{
    string lic_number=1;
    string name = 2;
    Gender gender = 3;
    int64 birth_date_millis = 4;
}
message GetProfileRequest{}
message ClearProfileRequest{}
service ProfileService{
    rpc GetProfile (GetProfileRequest) returns (Profile);
    rpc SubmitProfile (Identity) returns (Profile);
    rpc ClearProfile (ClearProfileRequest) returns (Profile);
}
```

`rental.yaml`配置

```go
type: google.api.Service
config_version: 3

http:
  rules:
    - selector: rental.v1.TripService.CreateTrip
      post: /v1/trip/createTrip
      body: "*"
    - selector: rental.v1.TripService.GetTrip
      get: /v1/trip/{id}
    - selector: rental.v1.TripService.GetTrips
      get: /v1/trips
    - selector: rental.v1.TripService.UpdateTrip
      put: /v1/trip/{id}
      body: "*"
    - selector: rental.v1.ProfileService.GetProfile
      get: /v1/profile
    - selector: rental.v1.ProfileService.SubmitProfile
      post: /v1/profile
      body: "*"
    - selector: rental.v1.ProfileService.ClearProfile
      delete: /v1/profile
```

# 用户服务实现

实现获取、提交、更新profile。

```go
package profile

type Service struct {
	Mongo  *dao.Mongo
	Logger *zap.Logger
}

func (s *Service) GetProfile(c context.Context, req *rentalpb.GetProfileRequest) (*rentalpb.Profile, error) {
	aid, err := auth.AccountIDFromContext(c)
	if err != nil {
		return nil, err
	}
	p, err := s.Mongo.GetProfile(c, aid)
	if err != nil {
		if err == mongo.ErrNoDocuments {
			return &rentalpb.Profile{}, nil
		}
		s.Logger.Error("cannot get Profile", zap.Error(err))
		return nil, status.Error(codes.Internal, "")
	}
	return p, nil
}
// 必须在UNSUBMITTED的条件下才可以提交
func (s *Service) SubmitProfile(c context.Context, req *rentalpb.Identity) (*rentalpb.Profile, error) {
	aid, err := auth.AccountIDFromContext(c)
	if err != nil {
		return nil, err
	}
	p := &rentalpb.Profile{
		Identity:       req,
		IdentityStatus: rentalpb.IdentityStatus_PENDING,
	}
	err = s.Mongo.UpdateProfile(c, aid, rentalpb.IdentityStatus_UNSUBMITTED, p)
	if err != nil {
		if err == mongo.ErrNoDocuments {
			return &rentalpb.Profile{}, nil
		}
		s.Logger.Error("cannot get Profile", zap.Error(err))
		return nil, status.Error(codes.Internal, "")
	}
	return p, nil
}
func (s *Service) ClearProfile(c context.Context, req *rentalpb.ClearProfileRequest) (*rentalpb.Profile, error) {
	aid, err := auth.AccountIDFromContext(c)
	if err != nil {
		return nil, err
	}
	p := &rentalpb.Profile{}
	err = s.Mongo.UpdateProfile(c, aid, rentalpb.IdentityStatus_VERIFIED, p)
	if err != nil {
		if err == mongo.ErrNoDocuments {
			return &rentalpb.Profile{}, nil
		}
		s.Logger.Error("cannot get Profile", zap.Error(err))
		return nil, status.Error(codes.Internal, "")
	}
	return p, nil
}

```

Dao层代码：

```go
package dao
const (
	accountIdField      = "accoundid"
	profileField        = "profile"
	identityStatusField = profileField + ".identityStatus"
)

type Mongo struct {
	col *mongo.Collection
}

func NewMongo(db *mongo.Database) *Mongo {
	return &Mongo{
		col: db.Collection("profile"),
	}
}

type ProfileRecord struct {
	AccountID string            `bson:"accoundid"`
	Profile   *rentalpb.Profile `bson:"profile"`
}

func (m *Mongo) GetProfile(c context.Context, aid id.AccountID) (*rentalpb.Profile, error) {
	res := m.col.FindOne(c, byAccountID(aid))
	if err := res.Err(); err != nil {
		return nil, err
	}
	var pr ProfileRecord
	err := res.Decode(&pr)
	if err != nil {
		return nil, fmt.Errorf("cannot decode profile record:%v", err)
	}
	return pr.Profile, nil

}

func (m *Mongo) UpdateProfile(c context.Context, aid id.AccountID, prevStatus rentalpb.IdentityStatus, p *rentalpb.Profile) error {
	_, err := m.col.UpdateOne(c, bson.M{
		accountIdField:      aid.String(),
		identityStatusField: prevStatus,
	}, mgo.Set(
		bson.M{
			accountIdField: aid.String(),
			profileField:   p,
		},
	), options.Update().SetUpsert(true))
	return err
}
func byAccountID(aid id.AccountID) bson.M {
	return bson.M{
		accountIdField: aid.String(),
	}
}
```

# 测试



# 联调

