---
author: "Brock Ramsey"
date: 2018-03-14
linktitle: URL Shortener in Golang
menu:
  main:
    parent: tutorials
title: URL Shortener in Golang
weight: 10
---
This is a basic URL shortener written in Golang, I write this with the assumption that the reader has basic programming knowledge. This is intended to simply demonstrate how I wrote a simple URL shortener in golang. This particular task is something I really enjoyed creating, this also brought back the spark for problem-solving. I've since gone on to several new projects since creating this. Ultimately, it needed a blog post, so here it is!

## Introduction
Let's declare a configuration file for handling some of our project data, we can then keep this data private from public repositories by simply committing a basic skeleton. This file would obviously contain our sensitive data. We will call it `app.json`.
```
{
    "host": "rootwire.space",
    "port": 8080,
    "salt": "pacificio looks like a turkey vulture",
    "dbuser": "youruser",
    "dbpass": "yourpass",
    "dbname": "yourdbname"
}
```
## Initializing Our Program
Starting off, we will decide what we're using for persistence. My persistence is setup using docker containers, I do recommend utilizing the same concept. In this particular application, we will be using `PostgreSQL` and `database/sql`. We will be using `viper` for handling configuration and `go-hashids` for generating the shortened url hash. Now lets start with declaring the main package and importing required packages.
```
package main

import (
    "database/sql"
    "fmt"
    "github.com/gin-gonic/gin"
    _ "github.com/lib/pq"
    "github.com/speps/go-hashids"
    "github.com/spf13/viper"
    "net/http"
    "net/url"
    "time"
)
```
## Validating User Input
What happens when the user inputs a url without an `http://` call? This requires some validation, we will create a function to vailidate user input and ensure the data is indeed starting with an `http://` call. This function will take the string and return a true or false boolean if the string exists or not.

```
func validateUrl(url_string string) bool {
    _, err := url.ParseRequestURI(url_string)
    if err != nil {
        return false
    } else {
        return true
    }
}
```
## The Main Function
This function can be broken down into individual functions for each route. However, this is a relatively simple project with few routes so we'll handle the routes in one main function. This function uses `viper` to grab our configuration data. We use `pq` to handle the database calls and `gin` for our route handling.

The first part of the main function handles the configuration data, the database connection and loads our html files.
```
func main() {
    viper.SetConfigName("app")
    viper.AddConfigPath(".")

    err := viper.ReadInConfig()

    if err != nil {
        panic(err)
    }

    domain := viper.GetString("host")
    port := viper.GetString("port")
    salt := viper.GetString("salt")
    dbuser := viper.GetString("dbuser")
    dbpass := viper.GetString("dbpass")
    dbname := viper.GetString("dbname")

    info := fmt.Sprintf("user=%s password=%s dbname=%s sslmode=disable", dbuser, dbpass, dbname)
    db, err := sql.Open("postgres", info)

    if err != nil {
        panic(err)
    }

    defer db.Close()

    router := gin.Default()

    router.LoadHTMLFiles("public/index.tmpl", "public/output.tmpl", "public/error.tmpl")
```
First, we handle our `GET` index `/` route.
```
    router.GET("/", func(c *gin.Context) {
        c.HTML(http.StatusOK, "public/index.tmpl", gin.H{
            "title": "rwshurl",
        })
    })
```
Here, we work our magic, we grab the url from the post form, we generate a hash for it, prepare a query statement and insert it to our database.
```
    router.POST("/s", func(c *gin.Context) {
        url := c.PostForm("url")

        if validateUrl(url) {
            hashdata := hashids.NewData()
            hashdata.Salt = salt
            hashdata.MinLength = 5

            h, _ := hashids.NewWithData(hashdata)

            now := time.Now()
            hash, _ := h.Encode([]int{int(now.Unix())})

            stmt, err := db.Prepare("INSERT INTO shortener (hash, url) VALUES ($1, $2)")

            if err != nil {
                panic(err)
            }

            _, err = stmt.Exec(hash, url)

            if err != nil {
                panic(err)
            }

            c.HTML(http.StatusOK, "public/output.tmpl", gin.H{
                "domain": domain,
                "title":  "rwshurl output",
                "hash":   hash,
                "url":    hash,
            })
        } else {
            c.HTML(http.StatusOK, "public/error.tmpl", gin.H{
                "error": "Url must contain http or https, please try again",
            })
        }
    })
```
What we do here is handle outputting our generated short URL. We grab our information from the database and display a link. Concluded with a redirect if the hash is called and exists in the database.
```
    router.GET("/s/:hash", func(c *gin.Context) {
        hash := c.Param("hash")
        stmt := `SELECT url from shortener where hash=$1`

        row := db.QueryRow(stmt, hash)
        switch err := row.Scan(&hash); err {
        case sql.ErrNoRows:
            fmt.Println("No rows returned!")
        case nil:
            c.Redirect(http.StatusMovedPermanently, hash)
        default:
            panic(err)
        }
    })

    router.Run(":" + port)
}
```
There we have it folks! That is my take on a simple URL shortener using `viper`, `postgresql` and `hashids`. Please feel free to comment with pointers and tips or even links to your own project designs. Any feedback is welcome, even negative, go ahead, I can take it!
