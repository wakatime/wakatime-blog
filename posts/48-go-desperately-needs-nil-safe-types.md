---
Title: Go Desperately Needs Nil Safe Types
Date: 2021-06-10
Image: https://wakatime.com/static/img/blog/go-nil-pointer-panic-issues.png
Description: Ways the Go language could fix nil pointer runtime panics.
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorGravatar: https://wakatime.com/gravatar/@alan
Category: Engineering
Tags: go
---

<img src="https://wakatime.com/static/img/blog/go-nil-pointer-panic-issues.png" alt="github issues cloud" style="width:100%" />

If you’ve worked with [Go][go lang] before, you’ve probably seen this runtime error.

    panic: runtime error: invalid memory address or nil pointer dereference

The current solution is checking `thevar != nil` before using the var, but forgetting to do that means your program will crash.
That means this simple programmer error could take down your whole server.
Hopefully you catch these errors with tests before they reach production.
However, we’re not machines so inevitably we won’t catch some of these until our end users encounter an error in production.


### Go should have types that can never be nil

[Jelte Fennema][jelte] already [proposed a solution][stream blog] over 3 years ago:

> The idea is really simple and is also used by Rust and C++: add a pointer type that can never be nil.

Jelte compares Go to Rust, saying that safe Rust doesn’t have nil-able pointers.

However, I think comparing Go to Swift fits better because Go will never get rid of nil pointers.

### Swift

Swift has two types of variables: [Optionals and Non-Optionals][swift].
Optional means nullable.

Non-optional variables in Swift are guaranteed to never be nil.
Your function no longer needs to check it’s arguments for nil if you declare your arguments non-optional.
Imagine deleting all those duplicate `if myarg == nil` in all your functions with the guarantee your inputs won’t ever be nil!

### Conclusion

Go2 has [several][proposal 28133] [open][proposal 22729] [proposals][proposal 30177] for non-nil types, but nothing seems to be decided upon yet.
Hopefully they don’t dismiss this lightly.
In my opinion, this is the only thing holding Go back from competing with Rust, Swift, TypeScript and other languages with non-nullable types.


[go lang]: https://golang.org/
[jelte]: https://keybase.io/jeltef
[stream blog]: https://getstream.io/blog/fixing-the-billion-dollar-mistake-in-go-by-borrowing-from-rust/#solution
[swift]: https://en.wikipedia.org/wiki/Swift_(programming_language)#Optionals_and_chaining
[proposal 28133]: https://github.com/golang/go/issues/28133
[proposal 22729]: https://github.com/golang/go/issues/22729
[proposal 30177]: https://github.com/golang/go/issues/30177
