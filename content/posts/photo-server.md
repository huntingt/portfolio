+++
title = "Photo Server"
date = "2021-12-05T10:30:49-05:00"
draft = false
+++

I wrote (and am still working on) this project primarily as a learning
experience. If I just wanted a server I would probably (and already have) just
wrote the server in javascript on node-js. The use of the sled KV database is
nice because it has a good Rust interface and is designed for stateful
services, and other embedded database bindings often miss features such as
transaction support. Sled, however, isn't 1.0 yet and doesn't have a guaranteed
file layout yet, and the serde KV database combination doesn't lend itself to
elegant database migrations. The intelligent choice would be to just use
something that supports SQL of course.

The system currently allows user creation, file upload, album creation, and
album sharing. The albums are based on Google Photo albums where photos are
grouped by day into sections which are lazily loaded into the frontend to allow
very large albums. Uploaded photos are thumbnailed by libvips into webp photos
(better compression than jpeg, and better support than AVIF). Videos are
thumbnailed using ffmpeg to grab the first frame of the video.

The frontend is a SPA built in solid-js, however, it currently lacks elegant
error handling and isn't pretty.
