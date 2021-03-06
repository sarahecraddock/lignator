# Input

- [inline](#inline)
- [file](#file)
- [directory](#directory)
- [header](#header)
- [template](#template)
- [tail](#tail)
- [variables](#variables)

When using lignator there are a few ways to set the template(s) it should use when running. You can use inline for passing a single template which allows you to generate a range of logs based on one template. You can pass a file which contains one or more templates. This is good if you want a simple way to create logs in different systems as part of a single run or a system that can output different formats into a single log file. The final approach is using a directory. This allows you to supply a whole directory of files which, like the previous example, can contain a single template or multiple templates per file. Check out the next section for more details on each.

## Inline

Inline is the simplist way to get started with lignator.You can simply pass a template inline and don't need to worry about creating templates and saving them to disk. We will cover the types of templates you can create later, but using the example we have seen already, we could run it like so:

```
$ lignator -t "[%{utcnow()}%] - [%{randomitem(INFO ,WARN ,ERROR)}%] - I am a log for request with id: %{uuid}%"
```

This, with the default options set, will output a single log into the output destination.

## File

You can supply template(s) to lignator via a file. It may be that you want to capture the template in source control or that you need to generate different logs into a single output in a similar way to some systems. Lets take the following as an example:

```
%{utcnow(MMM dd HH:mm:ss.ffffff)}% combo sshd(pam_unix)[%{randombetween(10000,30000)}%]: check pass; user %{linefromfile(./taxonomies/usernames.txt)}%
%{utcnow(MMM dd HH:mm:ss.ffffff)}% combo sshd(pam_unix)[%{randombetween(10000,30000)}%]: authentication failure; logname= uid=%{randombetween(0,255)}% euid=0 tty=NODEVssh ruser= rhost=%{randombetween(1,254)}%.%{randombetween(1,254)}%.%{randombetween(1,254)}%.%{randombetween(1,254)}%  user=%{linefromfile(./taxonomies/usernames.txt)}%
%{utcnow(MMM dd HH:mm:ss.ffffff)}% combo su(pam_unix)[%{randombetween(10000,30000)}%]: session opened for user %{linefromfile(./taxonomies/usernames.txt)}% by (uid=%{randombetween(0,255)}%)
%{utcnow(MMM dd HH:mm:ss.ffffff)}% combo su(pam_unix)[%{randombetween(10000,30000)}%]: session closed for user %{linefromfile(./taxonomies/usernames.txt)}%
%{utcnow(MMM dd HH:mm:ss.ffffff)}% combo sshd(pam_unix)[%{randombetween(10000,30000)}%]: authentication failure; logname= uid=%{randombetween(0,255)}% euid=%{randombetween(0,255)}% tty=NODEVssh ruser= rhost=%{randombetween(1,254)}%.%{randombetween(1,254)}%.%{randombetween(1,254)}%.%{randombetween(1,254)}%
%{utcnow(MMM dd HH:mm:ss.ffffff)}% combo logrotate: ALERT exited abnormally with [%{randombetween(1,255)}%]
```
If these were stored in a file called "linux.template" in the current directory you could run the following:

```
$ lignator -t ./linux.template -l 100
```

lignator would pick lines at random 100 times, transform the tokens and output the logs to the destination.

> While you would be guaranteed to get 100 lines, lignator is picking them at random so it is possible that not all the templates in the file will be used.

## Directory

Building on the previous examples, we could now build out a structure or templates for a range of systems and log types and have the logs easily generated as part of a single lignator execution. Let us take the following directory structure as an example:

> When working with a directory as the input, it will scan the supplied directory for all files with the extension .template

![Directory sturcture example](/images/examples-directory.png)

If this directory was stored within the current directory, we could run the following:

```
$ lignator -t ./templates -l 100
```

lignator would then randomly pick a file and then randomly pick a line from that file and transform it into a log until it had generated 100 logs into the output.

## Header

The header is a way to add something at the start of the file, this could be anything from static headers for a csv file or an input file with multiple lines and it's own tokens for transformation.

> Important note: When using the "--head" argument you can only pass it a template inline or via a specfic file

Here is a basic example for a csv file. This would generate a csv file with 2 lines, the first being the titles ID, Timestamp and Value. The second line would be the output of the tokens in the template.

```
$ lignator -H "ID,Timestamp,value" -t "%{uuid}%,%{utcnow()}%,%{randomitem(value1,value2)}%" -e csv
```

A more complex example could be a http request based on the following files and multiline inputs and outputs using the "-m" flag:

header.template

```
%{randomitem(POST,PUT)}% https://%{randombetween(1,254)}%.%{randombetween(1,254)}%.%{randombetween(1,254)}%.%{randombetween(1,254)}% HTTP/1.1
Content-Type: application/json

```

body.template

```
{
  "id": "%{uuid}%",
  "nested:" [
    {
      "%{utcnow}%"
    }
  ],
  "value": "%{randomitem(value1,value2)%"
}
```

```
$ lignator -H ./header.template -t body.template -e http -m true
```


## Template

The template is the core of lignator, this is where you can mix plaintext with tokens described in the documentation to generate most things you can imagine. Something simple like a CSV to something more complex like a http request, which we saw in the previous example.

## Tail

Just like the header argument we have alredy looked at the "--tail" argument is just the same, with the only difference being that it adds something to the end of the file rather than the start.

> Important note: When using the "--tail" argument you can only pass it a template inline or via a specfic file

## Variables

There are mainy cases where you may need to randomly generate a value and then re-use that value in a transformation of a template. To help with this we have added the ability to define variables. Setting them from the cli would look like:

```
$ lignator --variable myid=%{uuid}% --variable currenttime=%{utcnow()}%
```

As you can see right away, these are not statically defined values but are also in the form of a template which can contain one or more tokens. You can then use the variable token passing it the name of the variable you wish yo use.

```
$ lignator --variable myid=%{uuid}% -t "ID:%{variable(myid)}% - Yes the ID was: %{variable(myid)}%"
```

[>> Next - Output](/docs/output.md)