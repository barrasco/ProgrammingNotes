---
title: "VIP - View Interactor Presenter"
tags: "Planning,Programming"
---

## VIP Architecture

![image.png](https://boostnote.io/api/teams/MCXlxwtER/files/fd401fb207f68bcd94ddba24792d79d6226d34d690a318b708f256a807e55ae4-image.png)

* **View controller** - UI controller with strong reference to the Interator.
* **Interactor** - concrete class with API function call, and maintain data
business logic. Holds strong reference to Presenter.
* **Presenter** - Concrete class holds weak reference to View Controller.
Creates and passes Data to ViewController via ViewModels and validates data
and actions for ViewController.

### Flow:
1. `ViewController` takes user inputs and passes them to `Interactor` in form of requests.
2. `Interactor` processes these requests (validates and executes them) and passes the response to the `Presenter`.
3. `Presenter` processes these responses (validates and creates ViewModels) and passes them to `ViewController` to update/render UI.


[Reference 1](https://ayusinghi96.medium.com/vip-architecture-in-ios-7922b7da7c8e#:~:text=VIP%3A%20View%20Controller%2C%20Interactor%2C%20Presenter%20VIP%20is%20Uncle,and%20others.%20It%20consists%20of%20three%20core%20components%3A) |
[Reference 2](https://hackernoon.com/introducing-clean-swift-architecture-vip-770a639ad7bf) |
[Reference 3](https://www.softonitg.com/clean-swift-architecture/)
