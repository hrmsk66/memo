---
title: "Typeを知る"
date: 2020-03-09T11:24:31+09:00
draft: false
categories: ["post"]
tags: ["rust"]
---

1.38.0から追加された`std::any::type_name()`で型を調べることができる。  
Method chainingしてると扱っている型がなんだか分からなくなりがちなのでこういうのがあると助かる。  
https://doc.rust-lang.org/std/any/fn.type_name.html

```
fn print_type_name<T>(_val: T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let hello = "こんにちは";
    let world = "世界".to_string();
    
    print_type_name(hello);             // prints &str
    print_type_name(world.clone());     // prints alloc::string::String
    print_type_name(&world);            // prints &alloc::string::String
}
```