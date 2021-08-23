# Synchronous

### Python

In python, the most common way to handle errors in code would be with an if-statement like so:

```python
response = requests.get(URL)
if response.status_code == 200:
    content = response.content

    soup = bs.BeautifulSoup(content, 'lxml')
    articles = soup.find_all('article')
    if articles:
        ...
```

In itself, handling edge cases with  `if`  is not so bad and pretty natural. But, it has some drawbacks:

* It makes the code harder to read as it is difficult to dissociate business logic and error handling.
* if-statements are most of the time added in retroaction of a bug, which makes coding slower and not fun.
* if-statements error handling logic might vary between packages, which makes coding very tedious.

### Rust

In Rust, functions return either a success or an error, and, you have to deal with each case before compiling. This makes the code way more robust against errors at runtimes. In practice, the code might look like this:

```rust
let name = match node.find(Name("h3")).next() {
    Some(h3) => h3.text(),
    None => "".to_string(),
};
```

Rust also has other ways to handle Success / Errors Result, such as “?” à la Typescript:

```rust
let response = reqwest::get(&url).await?.text().await?;
```

The drawbacks are that developing in Rust is slower as you have to deal with every edge case during development.

Sometimes, it might even be tricky to understand which edge cases you’re trying to deal with. To be noted, that you HAVE to deal with every edge case in Rust otherwise, Rust will not compile. This is how serious Rust is about error handling.

