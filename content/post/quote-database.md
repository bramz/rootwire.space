---
author: "Brock Ramsey"
date: 2018-03-14
linktitle: Golang Quote Database
menu:
  main:
    parent: tutorials
title: Golang Quote Database
weight: 10
---
Here we assume the reader has a golang environment setup. Has basic programming knowledge in golang and knows how to install packages being used.

## Let's Get Started
First, let's decide what we're going to use for persistence. We will use SQLite in this guide, using a combination of `database/sql` and `go-sqlite3`. For routing purposes, we're going to use the gin web framework. So we start by declaring our package name and importing all required packages. We are using the `fmt` package to output errors here, this is not recommended, I used it to output errors simply for bug testing while developing. Starting off, we declare our package name, import packages we're going to use and set a variable for database.
```
package main

import (
    "database/sql"
    "fmt"
    "github.com/gin-gonic/gin"
    _ "github.com/mattn/go-sqlite3"
    "net/http"
)

var db *sql.DB
```
## The Main Function
This will be our main function using `gin` for routing and handling the `sqlite3` connection. Additionally, this will load the HTML templates being used for each page, we can use `LoadHTMLGlob()` as an alternative. Here, however, we will be using `LoadHTMLFiles()` to load our 4 basic templates. Followed by handlers for our `GET` and `POST` routes. Now, let's declare our function!
```
func main() {
    router := gin.Default()
    connect, err := sql.Open("sqlite3", "qdb.db")

    if err != nil {
        fmt.Println(err)
    }

    db = connect
    defer db.Close()

    router.LoadHTMLFiles("public/index.tmpl", "public/submit.tmpl", "public/search.tmpl", "public/output.tmpl")

    router.GET("/", randomQuote)
    router.GET("/submit", submitQuote)
    router.POST("/submit", submitQuote)
    router.GET("/search", searchQuote)
    router.POST("/search", searchQuote)
    router.POST("/delete/:id", deleteQuote)

    router.Run(":8080")
}
```
Note: Each template file should consist of `{{ define "path/to/template.tmpl" }} <html></html> {{ end}}`` to define the template.

## Our QDB Functionality
For a basic design, we will want to create functions to handle the following. Submitting quotes, displaying a random quote and searching quotes by `id`. Begin by declaring 3 functions for handling these actions.

We will start off with the submit quote function as we will need to submit quotes in order to fetch them for output. This function will output a form for the user to input quote data. After the user inputs the data and submits it we prepare a statement and execute the statement with our user quote as the value.
```
func submitQuote(context *gin.Context) {
    context.HTML(http.StatusOK, "public/submit.tmpl", gin.H{
        "header": "Submit Quote",
    })
    quote := context.PostForm("quote")

    stmt, err := db.Prepare("INSERT INTO quotes (quote) values (?)")
    if err != nil {
        fmt.Println(err)
    }

    _, err = stmt.Exec(quote)

    if err != nil {
        fmt.Println(err)
    }

}
```
Next, we want to display a random quote, this is also our index handler. This function selects a random quote from the database and outputs it to our index file.
```
func randomQuote(context *gin.Context) {
    rows, err := db.Query("SELECT id, quote FROM quotes ORDER BY RANDOM() LIMIT 1")
    if err != nil {
        panic(err)
    }

    var id int
    var quote string

    for rows.Next() {
        err = rows.Scan(&id, &quote)

        if err != nil {
            panic(err)
        }

        context.HTML(http.StatusOK, "public/index.tmpl", gin.H{
            "header": "Random Quotes",
            "id":     id,
            "quote":  quote,
        })
    }

}
```
Our last function will search out database by `id` and output the quote belonging to that `id`.
```
func searchQuote(context *gin.Context) {
    context.HTML(http.StatusOK, "public/search.tmpl", gin.H{
        "header": "Search Quotes",
    })

    id := context.PostForm("id")

    var created string
    var quote string

    stmt, err := db.Prepare("select created, quote from quotes where id = ?")

    if err != nil {
        panic(err)
    }

    defer stmt.Close()

    rows, err := stmt.Query(id)

    if err != nil {
        panic(err)
    }

    for rows.Next() {
        err := rows.Scan(&created, &quote)
        if err != nil {
            panic(err)
        }
        context.String(http.StatusOK, fmt.Sprintf("<p>id '%s'  created '%s' quote '%s'</p>", id, created, quote))
    }
}
```
One can add support for a delete function, however, I recommend a protection layer for that function in order to prevent deletions from unauthorized users. All in all, this concludes my guide to writing a quote database in golang. I hope you enjoy, please share and leave comments!

