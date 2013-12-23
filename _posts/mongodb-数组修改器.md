数组修改器

如果指定的键已存在，"$push"会向已有的数组末尾假如一个元素，要是没有会创建一个新的数组。

> \> db.blog.posts.find()

> { "_id" : ObjectId("52745af3f451a1c1be2465b2"), "title" : "A blog post", "content" : "..." }

> \> db.blog.posts.update({"title":"A blog post"},{$push:{"comments":{"name":"joe","email":"joe@example.com","content":"nice post."}}})
> 
> \> db.blog.posts.find()

> { "_id" : ObjectId("52745af3f451a1c1be2465b2"), "comments" : [ 	{ 	"name" : "joe", 	"email" : "joe@example.com", 	"content" : "nice post." } ], "content" : "...", "title" : "A blog post" }

> \> db.blog.posts.update({"title":"A blog post"},{$push:{"comments":{"name":"bob","email":"bob@exmaple.com","content":"good post."}}})
> 
> \> db.blog.posts.find()

>{ "_id" : ObjectId("52745af3f451a1c1be2465b2"), "comments" : [ 	{ 	"name" : "joe", 	"email" : "joe@example.com", 	"content" : "nice post." },	{ 	"name" : "bob", 	"email" : "bob@exmaple.com", 	"content" : "good post." } ], "content" : "...", "title" : "A blog post" }

