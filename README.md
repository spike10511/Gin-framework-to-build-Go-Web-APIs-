# 使用Gin框架构建Go Web API并通过前端与其交互

在这篇博客中，我们将使用Gin框架构建一个简单的图书管理系统，并通过一个基本的前端页面与API进行交互。我们将详细介绍项目结构、代码实现以及如何解决常见问题。

## 项目概述

我们将创建一个图书管理系统，它提供了基本的增删查改（CRUD）功能。用户可以通过API添加书籍、查看书籍列表、更新书籍信息以及删除书籍。然后，我们会构建一个简单的前端页面，通过该页面与API进行交互。

## 环境准备

首先，确保你已经在系统中安装了Go语言环境。如果没有，请前往[Go官方网站](https://golang.org/dl/)下载并安装。

### 初始化Go模块

在开始之前，我们需要初始化一个Go模块。打开终端并运行以下命令：

```bash
mkdir book-management
cd book-management
go mod init book-management
```

这将创建一个新的Go模块并生成go.mod文件。

### 安装Gin框架
接下来，我们需要安装Gin框架。运行以下命令：

```bash
go get -u github.com/gin-gonic/gin
```
如果你在安装Gin框架时遇到网络问题，可以设置Go代理为国内的镜像：

```bash
go env -w GOPROXY=https://goproxy.cn,direct
```

## 构建项目结构

我们将按照以下结构组织项目：

```bash
book-management/
├── static/
│   └── index.html
├── main.go
├── models/
│   └── book.go
├── controllers/
│   └── bookController.go
├── routers/
│   └── router.go
└── database/
    └── database.go
```

## 编写代码
main.go

```go
package main

import (
    "book-management/routers"
    "book-management/database"
    "github.com/gin-gonic/gin"
)

func main() {
    database.InitDB()

    router := gin.Default()

    // 提供静态文件
    router.Static("/static", "./static")

    // 初始化路由
    routers.InitRoutes(router)

    router.Run(":8080")
}
```

models/book.go

```go
package models

type Book struct {
    ID     uint   `json:"id"`
    Title  string `json:"title"`
    Author string `json:"author"`
    ISBN   string `json:"isbn"`
}

var Books []Book
```

controllers/bookController.go

```go
package controllers

import (
    "book-management/models"
    "github.com/gin-gonic/gin"
    "net/http"
    "strconv"
)

func GetBooks(c *gin.Context) {
    c.JSON(http.StatusOK, models.Books)
}

func AddBook(c *gin.Context) {
    var newBook models.Book
    if err := c.ShouldBindJSON(&newBook); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    newBook.ID = uint(len(models.Books) + 1)
    models.Books = append(models.Books, newBook)
    c.JSON(http.StatusOK, newBook)
}

func UpdateBook(c *gin.Context) {
    id, _ := strconv.Atoi(c.Param("id"))

    for i, book := range models.Books {
        if book.ID == uint(id) {
            if err := c.ShouldBindJSON(&models.Books[i]); err != nil {
                c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
                return
            }
            c.JSON(http.StatusOK, models.Books[i])
            return
        }
    }
    c.JSON(http.StatusNotFound, gin.H{"message": "Book not found"})
}

func DeleteBook(c *gin.Context) {
    id, _ := strconv.Atoi(c.Param("id"))

    for i, book := range models.Books {
        if book.ID == uint(id) {
            models.Books = append(models.Books[:i], models.Books[i+1:]...)
            c.JSON(http.StatusOK, gin.H{"message": "Book deleted"})
            return
        }
    }
    c.JSON(http.StatusNotFound, gin.H{"message": "Book not found"})
}
```
routers/router.go

```go
package routers

import (
    "book-management/controllers"
    "github.com/gin-gonic/gin"
)

func InitRoutes(router *gin.Engine) {
    router.GET("/books", controllers.GetBooks)
    router.POST("/books", controllers.AddBook)
    router.PUT("/books/:id", controllers.UpdateBook)
    router.DELETE("/books/:id", controllers.DeleteBook)
}
```
database/database.go

```go
package database

import (
    "log"
)

func InitDB() {
    log.Println("Database initialized (placeholder function)")
}
```

## 构建前端页面
接下来，我们将创建一个简单的前端页面来与我们的API进行交互。
static/index.html

在项目根目录下的 static 目录中创建一个 index.html 文件：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Book Management</title>
</head>
<body>
    <h1>Book Management</h1>

    <div>
        <h2>Add a Book</h2>
        <form id="addBookForm">
            <label for="title">Title:</label>
            <input type="text" id="title" name="title" required><br><br>
            <label for="author">Author:</label>
            <input type="text" id="author" name="author" required><br><br>
            <label for="isbn">ISBN:</label>
            <input type="text" id="isbn" name="isbn" required><br><br>
            <input type="submit" value="Add Book">
        </form>
    </div>

    <div>
        <h2>Books List</h2>
        <button id="fetchBooks">Fetch Books</button>
        <ul id="booksList"></ul>
    </div>

    <script>
        document.getElementById("addBookForm").addEventListener("submit", function(event) {
            event.preventDefault();
            const title = document.getElementById("title").value;
            const author = document.getElementById("author").value;
            const isbn = document.getElementById("isbn").value;

            fetch("http://localhost:8080/books", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json"
                },
                body: JSON.stringify({ title, author, isbn })
            })
            .then(response => response.json())
            .then(data => {
                alert("Book added successfully!");
            })
            .catch(error => {
                console.error("Error:", error);
            });
        });

        document.getElementById("fetchBooks").addEventListener("click", function() {
            fetch("http://localhost:8080/books")
            .then(response => response.json())
            .then(data => {
                const booksList = document.getElementById("booksList");
                booksList.innerHTML = "";
                data.forEach(book => {
                    const li = document.createElement("li");
                    li.textContent = `${book.title} by ${book.author} (ISBN: ${book.isbn})`;
                    booksList.appendChild(li);
                });
            })
            .catch(error => {
                console.error("Error:", error);
            });
        });
    </script>
</body>
</html>
```

## 运行并测试应用

#### 1.启动服务器

在终端中运行以下命令启动Gin服务器：

```bash
go run main.go
```
如果一切正常，你会看到类似以下内容的输出：
![图片](https://github.com/user-attachments/assets/2c8d581d-7260-4c5c-8d47-194807d953d4)




#### 2. 访问前端页面

在浏览器中访问 http://localhost:8080/static/index.html，你将看到一个简单的前端界面，可以通过该界面添加书籍并查看书籍列表。
![图片](https://github.com/user-attachments/assets/8bf02301-0f55-4097-bc85-e8789ef0276e)

然后你可以进入http://localhost:8080/books，这里会显示你添加的书目，在前端页面添加后再刷新此页面就能看到。
![图片](https://github.com/user-attachments/assets/dbd38f3b-57fb-410a-95f6-d33e474838c9)



--------------------------------------------------------



通过这个教程，你学会了如何使用Gin框架构建一个简单的Go Web API，并通过前端页面与API进行交互。如果你有更多疑问或需要进一步的帮助，请在评论区留言。


转载请注明出处。
