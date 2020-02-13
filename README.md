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

type Author {
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

        // subscription
        onNewBook() >> Book
    }
    authors: {
        find(uids: [string?]?): [Author?]
        findOne(uid: string): Author?
    }
}
```

```
// use 声明调用参数
// let 把值绑定到名称
// $ 调用服务

// 查询一本书的作者

use bookId in
let book = $books.findOne(bookId) in
if book == null then
    let author = $authors.findOne(book.authorId) in
    if author != null then
        author.name
    end
end

// 或者使用 ?> 管道操作符
// 如果 $books.findOne(bookId) 返回 null，则忽略后面的调用，直接返回 null

use bookId in
$books.findOne(bookId)
?> @select (book => book.authorId)
|> $authors.findOne
?> @select (author => author.name)
```

```
// => 符号用来描述一个数据映射
// @ 符号表示自定义的数据映射函数
// |> 管道操作符，可以在数据映射表达式/数据映射函数之间传值

// 根据 bookIds 批量查询对应的书名和作者

use bookIds in

let books = $books.find(bookIds) in
let authorIds = books |> @map (book => book?.authorId) in
let authors = $authors.find(authorIds) in
books |> @map ((book, index) => {
    bookName: book.name,
    author: authors[i]?.name
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
    let newPrices = books |> @map (book => {
        id: book.id
        newPrice: book.price * 0.8
    }) in
    $books.update(newPrices)
end
```

```
// 订阅指定作者的新书的书名

use authorId in
$books.onNewBook()
|> @filter (book => book.authorId = authorId)
|> @select (book => book.name)
```