# หัดใช้ MongoDB และลองเชื่อมต่อด้วย Go

เริ่มต้นด้วยการติดตั้ง MongoDB กันก่อน โดยในตัวอย่างนี้จะเป็นการติดตั้งบน macOS ด้วย Homebrew

สำหรับใครที่ยังไม่มี Homebrew ก็ติดตั้งก่อนดังนี้

	$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

จากนั้นทำการติดตั้ง mongo

	$ brew update
	$ brew install mongodb

```
Notes:

To have launchd start mongodb now and restart at login:
	brew services start mongodb
	 
Or, if you don't want/need a background service you can just run:
	mongod --config /usr/local/etc/mongod.conf
```

ลอง start mongo บนเครื่องของเราด้วยคำสั่ง

	$ mongod

จะพบว่าไม่สามารถ start MongoDB ได้ โดยมี error แจ้งว่ายังไม่มีไดเร็คทอรี่ `/data/db`
ซึ่งเป็น default path ที่ใช้ในการเก็บ data ของ MongoDB ถ้าเครื่องเรายังไม่มีไดเร็คทอรี่นี้อยู่ ทำให้ไม่สามารถ start ขึ้นมาได้ ดังนั้นให้สร้างไดเร็คทอรี่ขึ้นมาก่อน

	$ sudo mkdir -p /data/db

และกำหนดสิทธิให้ตัวเองเป็นเจ้าของโฟลเดอร์นี้

	$ sudo chown -Rv $USER /data/db

ลองสั่ง `mongod` อีกครั้ง จะสามารถใช้งาน mongo บน port `27017` ได้แล้ว

## Connect

ทำการ connect เข้าไปยัง mongodb ที่อยู่บนเครื่องของเรา

	$ mongo localhost


### Create Database

คราวนี้เรามาสร้าง database กันก่อนดีกว่า

	> use {DATABASE_NAME}

คำสั่ง use เป็น คำสั่งที่บอกว่าเราจะเข้าไปใช้ database ไหน แต่ถ้าไม่มี database นั้นอยู่ในเครื่องก็จะทำการสร้างให้เราเองและเข้าถึง database ที่เพิ่งสร้างขึ้นมาใหม่ให้โดยอัตโนมัติ

### Create

	db.{COLLECTION_NAME}.insertOne({DOCUMENT})

insertOne() เป็นคำสั่งที่ใช้ในการเพิ่มข้อมูลลงใน collection ที่เราต้องการ ซึ่งหากยังไม่มี collection นั้นก็จะทำการสร้าง collection ใหม่ให้โดยอัตโนมัติ

ตัวอย่าง

	> db.users.insertOne({name: 'Jun', lastName: 'Lee', age: 38})

	{
		"acknowledged" : true,
		"insertedId" : ObjectId("5d44f480642bf4a1c8cb22fe")
	}

### Read

	db.{COLLECTION_NAME}.find({FILTER})

find() เป็นคำสั่งที่ใช้ในการเรียกดูข้อมูลทั้งหมดใน collection ที่เราต้องการ

ตัวอย่าง

	> db.users.find()

	{ "_id" : ObjectId("5d44f480642bf4a1c8cb22fe"), "name" : "Jun", "lastName" : "Lee", "age" : 38 }
	{ "_id" : ObjectId("5d44f651642bf4a1c8cb22ff"), "name" : "John", "lastName" : "Doe", "age" : 20 }
	{ "_id" : ObjectId("5d44f666642bf4a1c8cb2300"), "name" : "Alice", "lastName" : "Bob", "age" : 15 }

เราสามารถเลือกใส่เงื่อนไขให้กับการค้นหาได้ เช่น ต้องการผลลัพทธ์ สำหรับผู้ใช้ที่ชื่อ Alice

	> db.users.find({name: 'Alice'})
	
	{ "_id" : ObjectId("5d44f666642bf4a1c8cb2300"), "name" : "Alice", "lastName" : "Bob", "age" : 15 }

ต้องการค้นหาผู้ใช้ที่มีอายุน้อยกว่า 20 ปี

	> db.users.find({age: {$lte :20}})
	
	{ "_id" : ObjectId("5d44f651642bf4a1c8cb22ff"), "name" : "John", "lastName" : "Doe", "age" : 20 }
	{ "_id" : ObjectId("5d44f666642bf4a1c8cb2300"), "name" : "Alice", "lastName" : "Bob", "age" : 15 }

ซึ่ง find() จะเป็นการเรียกดูผลลัพทธ์ทั้งหมดที่พบ หากเราต้องการดูแค่ผลลัพทธ์แรกที่พบ สามารถใช้ findOne() แทนได้

	> db.users.findOne({age: {$lte :20}})

	{
		"_id" : ObjectId("5d44f651642bf4a1c8cb22ff"),
		"name" : "John",
		"lastName" : "Doe",
		"age" : 20
	}

ดูเงื่อนไขสำหรับการคิวรี่ ได้ที่ [Comparison Query Operators](https://docs.mongodb.com/manual/reference/operator/query-comparison/)


### Update

	db.{COLLECTION_NAME}.updateMany({QUERY}, {UPDATE})

updateMany() เป็นคำสั่งที่ใช้แก้ไขข้อมูลทั้งหมดใน collection ที่ตรงกับเงื่อนไขที่เรากำหนดไว้ เช่น ต้องการเพิ่ม field ‘role’ ให้กับผู้ใช้ทุกคน

ตัวอย่าง

	> db.users.updateMany({}, { $set: {role: 'member'}})
	{ "acknowledged" : true, "matchedCount" : 3, "modifiedCount" : 3 }

	> db.users.find()
	
	{ "_id" : ObjectId("5d44f480642bf4a1c8cb22fe"), "name" : "Jun", "lastName" : "Lee", "age" : 38, "role" : "member" }
	{ "_id" : ObjectId("5d44f651642bf4a1c8cb22ff"), "name" : "John", "lastName" : "Doe", "age" : 20, "role" : "member" }
	{ "_id" : ObjectId("5d44f666642bf4a1c8cb2300"), "name" : "Alice", "lastName" : "Bob", "age" : 15, "role" : "member" }

ถ้าเราต้องการอัพเดตให้ John เป็น ‘admin’ สามารถใช้ findOneAndUpdate()

	> db.users.findOneAndUpdate({ name: 'John'}, { $set: { role: 'admin'}})

	{
		"_id" : ObjectId("5d44f651642bf4a1c8cb22ff"),
		"name" : "John",
		"lastName" : "Doe",
		"age" : 20,
		"role" : "member"
	}

ตรวจสอบผลการ update จะพบว่า role เปลี่ยนเป็น admin แล้ว

	> db.users.findOne({name: 'John'})
	
	{
		"_id" : ObjectId("5d44f651642bf4a1c8cb22ff"),
		"name" : "John",
		"lastName" : "Doe",
		"age" : 20,
		"role" : "admin"
	}

### Delete

	db.{COLLECTION_NAME}.findOneAndDelete({FILTER})

findOneAndDelete() เป็นคำสั่งในการค้นหาข้อมูลที่ตรงกับเงื่อนไข และลบข้อมูลนั้นทิ้งไป

	> db.users.findOneAndDelete({name: 'Alice'})

	{
		"_id" : ObjectId("5d44f666642bf4a1c8cb2300"),
		"name" : "Alice",
		"lastName" : "Bob",
		"age" : 15,
		"role" : "member"
	}
	
	> db.users.find()
	
	{ "_id" : ObjectId("5d44f480642bf4a1c8cb22fe"), "name" : "Jun", "lastName" : "Lee", "age" : 38, "role" : "member" }
	{ "_id" : ObjectId("5d44f651642bf4a1c8cb22ff"), "name" : "John", "lastName" : "Doe", "age" : 20, "role" : "admin" }


สามารถดูคำสั่งเพิ่มเติมโดยพิมพ์ `help` และหากต้องการออกจาก mongodb shell ให้พิมพ์ `exit`

## ทดสอบเชื่อมต่อ MongoDB ด้วย Go

ตัวอย่างโปรแกรม `main.go`

```go
package main

import (
	"context"
	"fmt"
	"log"

	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/bson/primitive"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)

//User type
type User struct {
	ID       primitive.ObjectID `bson:"_id,omitempty"`
	Name     string             `json:"name" bson:"name"`
	LastName string             `json:"lastName" bson:"lastName"`
	Age      int                `json:"age" bson:"age"`
	Role     string             `json:"role" bson:"role"`
}

func main() {

	clientOptions := options.Client().ApplyURI("mongodb://localhost:27017")
	
	client, err := mongo.Connect(context.TODO(), clientOptions)
	if err != nil {
		log.Fatal(err)
	}

	err = client.Ping(context.TODO(), nil)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("Connected to MongoDB!")

	collection := client.Database("test").Collection("users")

	// Pass these options to the Find method
	findOptions := options.Find()
	findOptions.SetLimit(2)

	// Here's an array in which you can store the decoded documents
	var results []*User

	// Passing bson.D{{}} as the filter matches all documents in the collection
	cur, err := collection.Find(context.TODO(), bson.D{{}}, findOptions)
	if err != nil {
		log.Fatal(err)
	}

	// Finding multiple documents returns a cursor
	// Iterating through the cursor allows us to decode documents one at a time
	for cur.Next(context.TODO()) {

		// create a value into which the single document can be decoded
		var elem User
		err := cur.Decode(&elem)
		if err != nil {
			log.Fatal(err)
		}

		results = append(results, &elem)
		fmt.Println(elem)
	}

	if err := cur.Err(); err != nil {
		log.Fatal(err)
	}

	// Close the cursor once finished
	cur.Close(context.TODO())
	fmt.Printf("Found multiple documents (array of pointers): %+v\n", results)

	for _, element := range results {
		fmt.Println(*element)
	}
```

### ทดสอบรันโปรแแกรม

	$ go run main.go
	
ผลลัพธ์

	Connected to MongoDB!
	Found multiple documents (array of pointers): [0xc00013e280 0xc00013e2d0]
	{ObjectID("5d44f480642bf4a1c8cb22fe") Jun Lee 38 member}
	{ObjectID("5d44f651642bf4a1c8cb22ff") John Doe 20 admin}

สามารถดูตัวอย่างเพิ่มเติมได้ การทำ CRUD Operations ได้จาก [Official MongoDB Go Driver Tutorial](https://www.mongodb.com/blog/post/mongodb-go-driver-tutorial)

## อ้างอิง

- [Install MongoDB Community Edition on macOS](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/)
- [บันทึกประสบการณ์ MongoDB First Time](https://engineering.thinknet.co.th/%E0%B8%9A%E0%B8%B1%E0%B8%99%E0%B8%97%E0%B8%B6%E0%B8%81%E0%B8%9B%E0%B8%A3%E0%B8%B0%E0%B8%AA%E0%B8%9A%E0%B8%81%E0%B8%B2%E0%B8%A3%E0%B8%93%E0%B9%8C-mongodb-first-time-2daa9150b558)
