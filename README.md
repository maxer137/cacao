# Cacao

This library provides safe Rust bindings for `AppKit` on macOS and (eventually) `UIKit` on iOS. It
tries to do so in a way that, if you've done programming for the framework before (in Swift or
Objective-C), will feel familiar. This is tricky in Rust due to the ownership model, but some
creative coding and assumptions can get us pretty far.

This library is currently _very_ early stage and may have bugs. Your usage of it is at
your own risk. With that said, provided you follow the rules (regarding memory/ownership) it's
already fine for some apps.

>_Note that this crate relies on the Objective-C runtime. Interfacing with the runtime **requires**
unsafe blocks; this crate handles those unsafe interactions for you and provides a safe wrapper, 
but by using this crate you understand that usage of `unsafe` is a given and will be somewhat 
rampant for wrapped controls. This does **not** mean you can't assess, review, or question unsafe 
usage - just know it's happening, and in large part it's not going away. Issues pertaining to the 
existence of unsafe will be closed without comment._

# Hello World

```rust
use cacao::app::{App, AppDelegate};
use cacao::window::Window;

#[derive(Default)]
struct BasicApp {
    window: Window
}

impl AppDelegate for BasicApp {
    fn did_finish_launching(&mut self) {
        self.window.set_minimum_content_size(400., 400.);
        self.window.set_title("Hello World!");
        self.window.show();
    }
}

fn main() {
    App::new("com.hello.world", BasicApp::default()).run();
}
```

For more thorough examples, check the `examples/` folder - for each UI control that's supported there will (ideally) be an example to accompany it.

## Initialization
Due to the way that AppKit and UIKit programs typically work, you're encouraged to do the bulk
of your work starting from the `did_finish_launching()` method of your `AppDelegate`. This
ensures the application has had time to initialize and do any housekeeping necessary behind the
scenes.

## Currently Supported
In terms of mostly working pieces, the following currently work:

- `App` initialization and event delegation
- `Window` construction, handling, and event delegation
- `View` construction, basic styling, some event delegation
- `ViewController` construction, lifecycle delegation
- `Toolbar` construction and basic API
- `WebView` with a basic API for handling callbacks
- `Autolayout` for View layout and such.

There are other components in this repository that exist as testbeds; they'll be added to this list as they reach some level of stability.

## Optional Features

The following are a list of [Cargo features][cargo-features] that can be enabled or disabled.

- **cloudkit**: Links `CloudKit.framework` and provides some wrappers around CloudKit
functionality. Currently not feature complete.
- **user-notifications**: Links `UserNotifications.framework` and provides functionality for
emitting notifications on macOS and iOS. Note that this _requires_ your application be
code-signed, and will not work without it.
- **webview**: Links `WebKit.framework` and provides a `WebView` control backed by `WKWebView`.
- **webview-downloading**: Enables downloading files from the `WebView` via a private
interface. This is not an App-Store-safe feature, so be aware of that before enabling.

[cargo-features]: https://doc.rust-lang.org/stable/cargo/reference/manifest.html#the-features-section

## General Notes
**Why not extend the existing cocoa-rs crate?**  
A good question. At the end of the day, that crate (I believe, and someone can correct me if I'm wrong) is somewhat tied to Servo, and I wanted to experiment with what the best approach for representing the Cocoa UI model in Rust was. This crate doesn't ignore their work entirely, either - `core_foundation` and `core_graphics` are used internally and re-exported for general use.

**Why should I write in Rust, rather than X language?**  
In _my_ case, I want to be able to write native applications for my devices (and the platform I like to build products for) without being locked in to writing in Apple-specific languages... and without writing in C/C++ or JavaScript (note: the _toolchain_, not the language - ES6/Typescript are fine). I want to do this because I'm tired of hitting a mountain of work when I want to port my applications to other ecosystems. I think that Rust offers a (growing, but significant) viable model for sharing code across platforms and ecosystems without sacrificing performance.

_(This is the part where the internet lights up and rants about some combination of Electron, Qt, and so on - we're not bothering here as it's beaten to death elsewhere)_

You'll like this crate if you like Rust. There is no perfect language, so write in whatever you think is best.

**Isn't Objective-C dead?**  
Yes, and no.

It's true that Apple definitely favors Swift, and for good reason (and I say this as an unabashed lover of Objective-C). With that said, I would be surprised if we didn't have another ~5+ years of support; Apple is quick to deprecate, but removing the Objective-C runtime would require a ton of time and effort. Maybe SwiftUI kills it, who knows. A wrapper around this stuff should conceivably make it easier to swap out the underlying UI backend whenever it comes time.

One thing to note is that Apple _has_ started releasing Swift-only frameworks. For cases where you need those, it should be possible to do some combination of linking and bridging - which would inform how swapping out the underlying UI backend would happen at some point.

Some might also decry Objective-C as slow. To that, I'd note the following:

- Your UI engine is probably not the bottleneck.
- Swift is generally better as it fixes a class of bugs that Objective-C doesn't catch; for the most part it still sits on top of the existing Cocoa frameworks anyway.
- Message dispatching in Objective-C is more optimized than significant chunks of the code you'll write, and is fast enough for most things.

**tl;dr** it's probably fine, and you have Rust for your performance needs.

**Why not just wrap UIKit, and then rely on Catalyst?**  
I have yet to see a single application where Catalyst felt good. The goal is good, though, and if it got to a point where that just seemed like the way forward (e.g, Apple just kills AppKit) then it's certainly an option.

**You can't possibly wrap all platform-specific behavior here...**  
Correct! Each UI control contains a `objc` field, which you can use as an escape hatch - if the control doesn't support something, you're free to drop to the Objective-C runtime yourself and handle it.

**Why don't you use bindings to automatically generate this stuff?**  
For initial exploration purposes I've done most of this by hand, as I wanted to find an approach that fit well in the Rust model before committing to binding generation. This is something I'll likely focus on next now that I've got things "working" well enough.

**Can I use this now?**  
For now, you can clone this repository and link it into your `Cargo.toml` by path. I'm squatting the names on `crates.io`, as I (in time) will throw this up there, but only when it's at a point where there's reasonable expectation that things won't be changing around much.

If you're interested in seeing this in use in a shipping app, head on over to [subatomic](https://github.com/ryanmcgrath/subatomic/).

**Is this related to Cacao, the Swift project?**  
No. The project referred to in this question aimed to map portions of Cocoa and UIKit over to run on Linux, but hasn't seen activity in some time (it was really cool, too!).

Open source project naming in 2020 is like trying to buy a `.com` domain: everything good is taken. Luckily, multiple projects can share a name... so that's what's going to happen here.

**Isn't this kind of cheating the Rust object model?**  
Depends on how you look at it. I personally don't care too much - the GUI layer for these platforms is a hard requirement to support for certain classes of products, and giving them up also means giving up battle-tested tools for things like Accessibility and deeper OS integration. With that said, internally there are efforts to try and make things respect Rust's model of how things should work.

You can think of this as similar to gtk-rs. If you want to support or try a more _pure_ model, go check out Druid or something. :)

## License
Dual licensed under an MIT/MPL-2.0 license. See the appropriate files in this repository for more information. Apple, AppKit, UIKit, Cocoa, and other trademarks are copyright Apple, Inc.

## Questions, Comments, etc
You can follow me over on [twitter](https://twitter.com/ryanmcgrath/) or [email me](mailto:ryan@rymc.io) with questions that don't fit as an issue here.
