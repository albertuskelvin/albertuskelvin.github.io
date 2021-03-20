---
title: 'Installing and Executing Rust Code via Docker'
date: 2019-08-13
permalink: /posts/2019/08/run-rust-docker/
tags:
  - rust
  - docker
---

The best way to try new technologies without having clutter? Docker.

This article is about how to install and run Rust code via Docker.

<ul>
  <li>Create a new directory in the host machine. Let's call it "my-rust-app"</li>
  <li>Create a very simple <i>Dockerfile</i> with the following lines

<pre>
# pull the latest version of Rust
FROM rust:latest

# change our work dir to /usr/myrustapp
WORKDIR /usr/myrustapp

# copy all the files in current dir to the work dir
COPY . .

# compile and run Rust script
CMD ["cargo", "run"]
</pre>

</li>

<li>Create a file called <b>Cargo.toml</b> and pass the following lines to the file.

<pre>
[package]
name = "hello_world"
version = "0.0.0"
authors = ["Your Name"]
edition = "2019"
</pre>

</li>

<li>Create a directory <b>src</b> and a file <b>main.rs</b> within it. Here's the content of the <b>main.rs</b>.

<pre>
fn main() {
    println!("Hello, World!");
}
</pre>

</li>

<li>Open up a Terminal and execute the following commands.

<pre>
# build the image from the docker file
docker build -t my-rust-img [path_to_the_dockerfile]

# create and run a container from the image
docker run -it --rm --name my-rust-container my-rust-img
</pre>

</li>

<li>The Rust script should be compiled and executed</li>
</ul>

Thanks for reading.
