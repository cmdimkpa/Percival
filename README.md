<img src='https://avariz-cdn.s3.us-east-2.amazonaws.com/percival-logo.png' width='200' />

# Percival
Percival is a **macro programming language** for performing ETL tasks.

- Percival is written in Python. 

- Percival simplifies ETL tasks by allowing you to write a complex ETL procedure as a simple, straight-forward script consisting of a number of `macros` - where each macro describes a group of related `operations`. 

- All the libraries you need for your ETL are loaded behind the scenes during runtime and you can invoke a particular functionality in the context of a `system macro` call. You can also evaluate pure Python expressions using the `eval` interface, or invoke the `shell/terminal` to run a command.

- As a language, Percival supports `procedural`, `declarative` and `functional` programming styles. Functional programming in Percival is achieved through the use of Python's `lambda` syntax. 

## Running Percival (pcv) scripts

```shell
$ python Percival script.pcv
```
## Hello, World!

```pcv
{
  ? Hello, World!;
}
```

## Examples

See some live ETL examples done with Percival in the `examples` folder.

## Inquiries

For questions and comments about Percival, contact: cmdimkpa@gmail.com
