# Tokens

- [uuid](#uuid)
- [randomitem](#randomitem)
- [randombetween](#randombetween)
- [linefromfile](#linefromfile)
- [utcnow](#utcnow)

The current tokens available are shown here:

| Token         | Description                                  | Example                                                          |
| ------------- | -------------------------------------------- | ---------------------------------------------------------------- |
| uuid          | Creates a new uuid per log                   | lignator -t "request id: %{uuid}%"                                 |
| randomitem    | Picks a random item from those provided      | lignator -t "LogLevel: %{randomitem(INFO , WARN , ERROR)}%"        |
| randombetween | Picks a random number between those provided | lignator -t "A Number between 1 and 10 = %{randombetween(1,10)}%"   |
| linefromfile  | Picks a random item from a file              | lignator -t "linefromfile: %{linefromfile(filepath)}%"             |
| utcnow        | Generate timestamp using UTC                 | lignator -t "timestamp: %{utcnow()}%"                              |
| variable      | Uses the result of a already transformed template so you can re use it per output | lignator -V myID=%{uuid}% -t "ID:%{variable(myID)}% the same ID: %{variable(myID)}%" |

## uuid

The uuid token is used to create a new unique id.

![uuid generation example](/images/lignator-uuid.gif)

## randomitem

The randonitem token can take a group of values inline, so it's good for when you are only working with a handful of items. Here is an example of generating logs with different log levels:

> Note in this example the space between some of the items is to ensure padding of the values in the log.

![randomitem example](/images/lignator-randomitem.gif)

## randombetween

The randombetween token allows lignator to generate a random number between and inclusive of the provided numbers. So in the example below it will create a number that is greater than 0 and less than 11.

```
$ lignator -t %{randombetween(1, 10)}%
```

![randombetween example](/images/lignator-randombetween.gif)

## linefromfile

The linefromfile token will let lignator pick at random a line from the supplied file. This is helpful when you have so many items that the randomitem becomes challegening to maintain or consume. A great example could be a file filled with browser agents.

![linefromfile example](/images/lignator-linefromfile.gif)

## utcnow

The utcnow token allows lignator to generate a range of different timestamps based on the current UTC date and time. Under the hood, it currently uses the .NET implementation of DateTime so you can use the details [here](https://docs.microsoft.com/en-us/dotnet/standard/base-types/standard-date-and-time-format-strings) for examples of all the available format strings.

> The utcnow function has an optional parameter allowing you to set the output format. By default it is "yyyy-MM-dd HH:mm:ss.fff"

Here are a few examples:

```
$ lignator -t "%{utcnow(yyyy-MM-dd)}%"
```

2021-02-10

```
$ lignator -t "%{utcnow(yyyy-MM-dd'T'HH:mm:ss.ffffff'Z')}%"
```
2021-02-10T04:32:50.540265Z

![utcnow example](/images/lignator-utcnow.gif)

## variable

The variable token allows lignator to evaluate a specific template and store the result to re use multiple times in a single log or output, this could be handy for when you want to output the same value in something like a firewall log or include the same IP in the url and host values of a http request header.

> Keep in mind that when lignator is in its default mode or transforming single line templates the variable will be in scope per line, when in multiline mode the variable will be in scope across lines but will within the scope of a single template transformation.

Here are a few examples:

```
$ lignator -V myID=%{uuid}% -t "MyID: %{variable(myID)}% Your ID isn't: %{variable(myID)}%" -l 10
```

You can see here that the id is re-used per line but then the variable is re-evaluated per log or output.

```
MyID: 5aaee6ef-35c8-4be3-b870-9439c3d03313 Your ID isn't: 5aaee6ef-35c8-4be3-b870-9439c3d03313
MyID: ea392ee7-1845-4b2d-9535-96a86072c1a8 Your ID isn't: ea392ee7-1845-4b2d-9535-96a86072c1a8
MyID: 79c5f880-5f80-465d-aaa3-1cfc19aa213d Your ID isn't: 79c5f880-5f80-465d-aaa3-1cfc19aa213d
MyID: 2e0cb86e-619f-4e53-b19b-34cf83e39294 Your ID isn't: 2e0cb86e-619f-4e53-b19b-34cf83e39294
MyID: 258f513a-7550-48bf-8fe3-73fc0c5ccd7b Your ID isn't: 258f513a-7550-48bf-8fe3-73fc0c5ccd7b
MyID: f066def1-a825-4fd7-90ae-dcd7253562aa Your ID isn't: f066def1-a825-4fd7-90ae-dcd7253562aa
MyID: 4e82f985-d76c-4b51-814f-28a98f495ef4 Your ID isn't: 4e82f985-d76c-4b51-814f-28a98f495ef4
MyID: 255339da-1724-4965-a0ed-1def22719f31 Your ID isn't: 255339da-1724-4965-a0ed-1def22719f31
MyID: 59a381b7-9126-48d2-b298-6c1f10d3acae Your ID isn't: 59a381b7-9126-48d2-b298-6c1f10d3acae
MyID: aa529347-8167-4068-acd5-3587d6916e52 Your ID isn't: aa529347-8167-4068-acd5-3587d6916e52
```

Multiline example

Using a input file like:

```
{
  "myID":"%{variable(myID)}%",
  "child": {
    "parentId":"%{variable(myID)}%"
  }
}
```

> the key here being the multiline argument being set to true

```
$ lignator -V myID=%{uuid}% -t ./variableexample.json -m true -e json
```

Generates

```
{
  "myID":"1f68bb63-bebb-44a4-b442-cf288e8d07e3",
  "child": {
    "parentId":"1f68bb63-bebb-44a4-b442-cf288e8d07e3"
  }
}
```

[>> Next - Considerations](/docs/considerations.md)