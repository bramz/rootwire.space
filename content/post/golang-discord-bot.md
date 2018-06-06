---
author: "Brock Ramsey"
date: 2018-06-06
linktitle: Simple Golang Discord Bot
menu:
  main:
    parent: tutorials
title: Simple Golang Discord Bot
weight: 10
---
I assume the reader is familiar with discord and has basic golang experience. Here we're going to use a pre existing package to handle the discord API. This saves us the trouble of "reinventing the wheel" in terms of handling the API.

## Before We Get Started
Before we get started, I want to recommend starting your own discord server. This will allow you to utilize all aspects of discord servers with a custom bot, with no restraints. After you have created your own server, you will need to grant authorization for your bot application. To do so, visit the [discord developer docs](https://discordapp.com/developers) and generate an auth token. We will be using this token to complete our bot to discord hand shake.

## Prerequisites And Our Bot Skeleton
Prior to starting our bot skeleton, readers may want to familiarize themselves with the following. We are going to be using the `discordgo` package for golang bindings with discord. More information can be found here: <https://github.com/bwmarrin/discordgo>
```
package main

import (
    "fmt"
    "github.com/bwmarrin/discordgo"
)

// our main function
func main() {
}

// our command handler function
func handleCmd() {
}
```
## Establishing Discord Session
We will want to establish a connection with discord, in order to handle session data. We will do so using our auth token, generated using the authorized apps for discord developers.
```
func main() {
    d, err := discordgo.New("Bot <your token here>")

    if err != nil {
        fmt.Println("failed to create discord session", err)
    }

    bot, err := d.User("@me")

    if err != nil {
        fmt.Println("failed to access account", err)
    }

    d.AddHandler(handleCmd)
    err = d.Open()

    if err != nil {
        fmt.Println("unable to establish connection", err)
    }

    defer d.Close()

    <-make(chan struct{})
}
```
## Handling Commands
Now we can use our command handler function to parse commands passed to the bot. The bot will respond accordingly when a command has been triggered. For an example instance, we will be parsing the `!test` command.
```
func handleCmd(d *discordgo.Session, msg *discordgo.MessageCreate) {
    user := msg.Author
    if user.ID == bid || user.Bot {
        return
    }

    content := msg.Content

    if (content == "!test") {
        d.ChannelMessageSend(msg.ChannelID, "Testing..")
    }

    fmt.Printf("Message: %+v\n", msg.Message)
}
```

That sums it up! I hope this basic example will allow beginners to expand from here, in hopes to create a custom bot for their own usage. Enjoy!
