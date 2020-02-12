# FunQL
Functional Query Language

```
// Schema
type Book {
    id: string
    name: string
    authorId: string
    price: number
}

type User {
    id: string
    name: string
}

type BookUpdateInput {
    id: string
    name: string?
    authorId: string?
    price: number?
}

{
    books: {
        find(bookIds: [string?]?): [Book?]
        findOne(bookId: string): Book?
        update(books: [BookUpdateInput]): void
    }
    users: {
        find(uids: [string?]?): [User?]
        findOne(uid: string): User
    }
}
```

```
// 查询一本书的书名和作者
//
// ? 符号声明客户端的调用参数
// $ 符号表示调用服务
// let 把值绑定到名称。只能绑定值类型，不能绑定表达式或者函数

?bookId: string

let book = $books.findOne(?bookId)
if (book != null) {
    let user = $users.findOne(book.authorId)
    {
        bookName: book.name,
        author: user.name
    }
}
```

```
// 根据 bookIds 批量查询对应的书名和作者
//
// => 符号用来描述一个简单数据映射表达式，不能调用服务
// @ 符号表示自定义的数据映射函数。只能在简单数据之间作映射，不能调用服务
// |> 管道操作符，可以在数据映射表达式/数据映射函数之间传值

?bookIds: [string]

let books = $books.find(?bookIds)
let authorIds = books |> @map (book => book?.authorId)
let users = $users.find(authorIds)
books |> @map ((book, index) => {
    bookName: book.name,
    author: users[i]?.name
})
```

```
// 所有书打八折
//
// ! 符号定义一个区块

!transaction {
    let books = $books.find(bookIds = null)
    let discountPrices = books |> @map (book => {
        id: book.id
        price: book.price * 0.8
    })
    $books.update(discountPrices)
}
```