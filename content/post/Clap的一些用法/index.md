+++
date = '2025-02-12T20:40:22+08:00'
draft = false
title = 'Clap的一些用法'
image = "navigation.svg"
slug = "Clap的一些用法"
description = "用Clap快速编写命令行程序"
categories = ["Code"]
tags = ["Clap", "Code", "Tools", "Rust"]
+++

Rust的 [Clap Crate](https://crates.io/crates/clap) 非常适用于开发命令行程序，它是一个“简单易用、高效且功能齐全的命令行参数解析器”。

Clap的作用在于帮助我们定义命令行工具的各种指令和用法，并解析用户使用的什么指令。之后我们只需要编写对应指令的功能代码即可。

在这里简单记录一下它的一些用法。
- crates.io: [link](https://crates.io/crates/clap)
- docs.rs: [link](https://docs.rs/clap/4.5.29/clap/)
- repo: [link](https://github.com/clap-rs/clap)
- Exapmles: [link](https://github.com/clap-rs/clap/tree/master/examples)

本文编写时：
```toml
clap = "4.5.29"
```

# Clap的两种API

Clap目前可以使用这两种方式构建命令行程序：
- Builder API
- Derive API
官方的example多数都提供了两种方式的示例。

若要使用Derive API，记得要添加feature：
```toml
clap = { version = "4.5.28", features = ["derive"] }
```

两种构建方式的区别主要是：
> Builder API
- **链式调用**的方式构建程序。
- 由于是 **链式调用**，代码 **很长很复杂**，可复用度较低。但在命令行参数需要经常变动调整时比较方便，加减几个函数调用就行。

> Derive API
- **声明式**，通过定义命令和参数对应的 `Strut` 和 `Enum` 并在上面使用 **属性宏** 构建程序。
- 使用 **声明式** 构建，代码 **直观简洁**，方便理解和维护。适合参数和命令比较稳定的命令行程序。

使用 **Derive API** 可以兼顾开发效率和可维护性，同时上手方便。如果有复杂定制或控制更多细节的需求，再使用Builder API。
在特定情况下，Derive 和 Builder API 可以混合使用： [Mixing Builder and Derive APIs](https://docs.rs/clap/4.5.29/clap/_derive/index.html#mixing-builder-and-derive-apis)


# 使用方式

> 官方给的Git示例比较好，推荐查看：
- [git-Builder API](https://github.com/clap-rs/clap/blob/master/examples/git.rs)
- [git-Derive API](https://github.com/clap-rs/clap/blob/master/examples/git-derive.rs)


## Builder API

Builder API 下，我们需要创建一个Command，通过调用 `get_matches()`函数来解析参数并获得 `ArgMathces`，然后对它进行模式匹配解析出相应指令，最后编写其对应的功能代码完成程序：
```rust
use clap::{Command, arg, Arg};

fn main() {
    let cmd = Command::new("CLI Name")
        .version("1.0")
        .author("Name <email@email.com>")
        .about("Introduction")
        .subcommand_required(true)
        // 位置参数
        .arg(
            Arg::new("input")
                .help("文件路径")
                .required(true)
        )
        // 选项参数
        .arg(
            Arg::new("name")
                .short('n')
                .long("name")
                .help("指定用户名")
                .arg_required_else_help(true)  
        )
        // 定义子命令
        .subcommand(
	        Command::new("diff")
                .about("比较两个文件")
                .arg(
	                Arg::new("full")
		                .help("全文比较")
		                .long("full")
		        )
		        .arg(
	                Arg::new("info")
		                .help("比较文件信息")
		                .long("info")
		        )
        )
    );
    let matches = cmd.get_matches();

	// 匹配参数
    let input = matches.value_of("input").unwrap();
    println!("输入文件: {}", input);
    
    if let Some(name) = matches.value_of("name") {
        println!("用户名: {}", name);
    } else {
        println!("未指定用户名");
    }

	// 匹配子命令
	match matches.subcommand {
		Some(("diff", diff_matches)) => {
			let full = diff_matches.get_one::<String>("full").map(|s| s.as_str());
			TODO("这里还是看官方Example好一些");
		}
	}
	
}

```


## Derive API

Derive API 下，我们需要创建一个结构体表示命令行程序，随后可以向其中嵌套添加结构体或枚举表示子命令、参数等：
```rust
use clap::Parser;

#[derive(Parser)]
#[command(next_line_help = true)]
#[command(name = "hello", version = "1.0", about = "一个简单的示例程序")]
struct Cli {
	// Option<T> 表示可选参数
    name: Option<String>,
    // 表示包含子命令。子命令定义在Commands枚举中
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]  
enum Commands {
	#[command(about = "A subcommand")]
	A,
	#[command(about = "B subcommand with args")]
	B(BArgs),
}

#[derive(Args)]  
struct BArgs {
	// 定义参数，value_enum 表示该参数的值是可选的几个给定值
	#[arg(short, long, required = true, value_enum)]  
	#[arg(help = "Info mode")]  
	mode: BArgsMode,
}

#[derive(Copy, Clone, ValueEnum)]  
enum InfoFaArgsMode {
	#[value(help = "aa")]  
	AA,
	#[value(help = "aa")]  
	BB,
}

fn main() {
	let args = Cli::parse();
	// 匹配可选参数name
	let name = args.name.unwrap();
	//匹配定义的command 子命令
	match args.command {
		Commands::A => { TODO() },
		Commands::B(args) => { TODO() },
	}	
}

```

# 常用配置

## 参数和子命令的一些设置

- 位置参数
结构体中不加 `#[arg(short, long)]` 时默认作为位置参数。位置参数可以直接使用：
```shell
my-app my_arg
```
- 选项参数
使用 `#[arg(short, long)]` 时作为选项参数，用户可以通过短选项（如 `-n`）或长选项（如 `--name`）传递参数：
```shell
my-app -a arg --barg arg
```
- 标志参数
当值的类型为布尔值时如 `debug: bool` 为标志参数，出现即为true,未出现为false。一般出现时与`action`联用（` #[arg(short, long, action = clap::ArgAction::Count)]`）：
```shell
my-app --debug
```


## 对Command的配置
- **name**
- **version**
- **author**
- **about / long_about**
- **propagate_version**
    - 如果设置为 true，子命令也会显示顶级命令的版本信息。
    - `#[command(propagate_version = true)]`
- **disable_help_flag / disable_version_flag**
    - 禁用时不会自动生成 `--help` 和 `--version`。
- **next_line_help**
    - 设为true时，帮助信息将会显示在命令名称的下一行。
- **term_width**
    - 指定帮助信息的输出宽度，便于美化输出。
    - `#[command(term_width = 80)]`
- allow_external_subcommands
- **help_template**
	- 自定义帮助信息模板，如下所示：
```rust
#[derive(Parser)]
#[command(
    about = "帮助信息定制示例",
    help_template = "\
{name} {version}
{about}

USAGE:
    {usage}

{all-args}
",
    term_width = 80
)]
```


对subcommand的配置与此类似。

## 对Args的配置

- **short 和 long**
    - 设置参数的短选项和长选项。
    - 示例：`#[arg(short, long)]`（自动推导名称），也可以显式设置：`#[arg(short = 'n', long = "name")]`
- **help**
- **value_name**
    - 指定参数在帮助信息中显示的值名。
    - 示例：`#[arg(value_name = "FILE")]`
- **default_value / default_value_t**
    - 设置默认值，可以是字符串或类型 T 的默认值。
    - 示例：`#[arg(default_value = "8080")]` 或 `#[arg(default_value_t = 8080)]`
- **required / required_if / required_if_eq**
    - 控制参数是否必填，或在某些条件下必填。
    - 示例：`#[arg(required = true)]` 或 `#[arg(required_if_eq("mode", "a"))]`
- **env**
    - 没有输入参数时将从设定的环境变量中读取值作为该参数。
    - `#[arg(env = "MYAPP_CONFIG")]`
- **value_parser**
    - 设置参数的解析器，可以指定类型转换、范围限制或自定义验证函数。
    - 示例：`#[arg(value_parser = clap::value_parser!(u16).range(1..=65535))]`
- **action**
    - 针对布尔值或计数参数，可以指定行为，例如 Count（出现一次该参数则值增长1）或 SetTrue（只要参数出现出现就设为 true）。
    - `#[arg(short, long, action = clap::ArgAction::Count)]`
- **conflicts_with / requires**
    - 用于定义参数之间的互斥或依赖关系。
    - 示例：`#[arg(conflicts_with = "other_arg")]` 或 `#[arg(requires = "config")]`
- **group**
    - 为参数设置组，同一个组的参数有一个被输入即可。
    - `#[arg(group = "input")]`
- **alias / short_alias / long_alias**
