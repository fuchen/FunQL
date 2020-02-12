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
    newName: string?
    newAuthorId: string?
    newPrice: number?
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
// use 声明调用参数
// let 把值绑定到名称
// $ 调用服务

// 查询一本书的书名和作者

use bookId in
let book = $books.findOne(bookId) in
if book then
    let user = $users.findOne(book.authorId);
    {
        bookName: book.name,
        author: user.name
    }
end
```

```
// => 符号用来描述一个数据映射
// @ 符号表示自定义的数据映射函数
// |> 管道操作符，可以在数据映射表达式/数据映射函数之间传值

// 根据 bookIds 批量查询对应的书名和作者

use bookIds in

let books = $books.find(bookIds) in
let authorIds = books |> @map (book => book?.authorId) in
let users = $users.find(authorIds) in
books |> @map ((book, index) => {
    bookName: book.name,
    author: users[i]?.name
})
```

```
// 定义一个区块:
// begin <block type> with
//     <block body>
// end

// 所有书打八折

begin transaction with
    let books = $books.find(null) in
    let newPrices = books |> map (book => {
        id: book.id
        newPrice: book.price * 0.8
    }) in
    $books.update(newPrices)
end
```