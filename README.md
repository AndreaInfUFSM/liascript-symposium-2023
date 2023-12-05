<!--
author:   Andrea Charão

email:    andrea@inf.ufsm.br

version:  0.0.1

language: EN

narrator: US English Female

comment: LiaScript Symposium 2023

translation: English  translations/English.md

link:     https://cdn.jsdelivr.net/chartist.js/latest/chartist.min.css

script:   https://cdn.jsdelivr.net/chartist.js/latest/chartist.min.js

script:   https://cdn.rawgit.com/davidshimjs/qrcodejs/gh-pages/qrcode.min.js


onload
window.CodeRunner = {
    ws: undefined,
    handler: {},

    init(url) {
        this.ws = new WebSocket(url);
        const self = this
        this.ws.onopen = function () {
            self.log("connections established");
            setInterval(function() {
                self.ws.send("ping")
            }, 15000);
        }
        this.ws.onmessage = function (e) {
            // e.data contains received string.

            let data
            try {
                data = JSON.parse(e.data)
            } catch (e) {
                self.warn("received message could not be handled =>", e.data)
            }
            if (data) {
                self.handler[data.uid](data)
            }
        }
        this.ws.onclose = function () {
            self.warn("connection closed")
        }
        this.ws.onerror = function (e) {
            self.warn("an error has occurred => ", e)
        }
    },
    log(...args) {
        console.log("CodeRunner:", ...args)
    },
    warn(...args) {
        console.warn("CodeRunner:", ...args)
    },
    handle(uid, callback) {
        this.handler[uid] = callback
    },
    send(uid, message) {
        message.uid = uid
        this.ws.send(JSON.stringify(message))
    }
}

//window.CodeRunner.init("wss://coderunner.informatik.tu-freiberg.de/")
//window.CodeRunner.init("wss://ancient-hollows-41316.herokuapp.com/")
window.CodeRunner.init("wss://java-coderunner.andreaschwertne.repl.co/")
//window.CodeRunner.init("ws://127.0.0.1:8000/")

@end


@LIA.c:       @LIA.eval(`["main.c"]`, `gcc -Wall main.c -o a.out`, `./a.out`)
@LIA.clojure: @LIA.eval(`["main.clj"]`, `none`, `clojure -M main.clj`)
@LIA.cpp:     @LIA.eval(`["main.cpp"]`, `g++ main.cpp -o a.out`, `./a.out`)
@LIA.go:      @LIA.eval(`["main.go"]`, `go build main.go`, `./main`)
@LIA.haskell: @LIA.eval(`["main.hs"]`, `ghc main.hs -o main`, `./main`)
@LIA.java:    @LIA.eval(`["@0.java"]`, `javac @0.java`, `java @0`)
@LIA.julia:   @LIA.eval(`["main.jl"]`, `none`, `julia main.jl`)
@LIA.mono:    @LIA.eval(`["main.cs"]`, `mcs main.cs`, `mono main.exe`)
@LIA.nasm:    @LIA.eval(`["main.asm"]`, `nasm -felf64 main.asm && ld main.o`, `./a.out`)
@LIA.python:  @LIA.python3
@LIA.python2: @LIA.eval(`["main.py"]`, `python2.7 -m compileall .`, `python2.7 main.pyc`)
@LIA.python3: @LIA.eval(`["main.py"]`, `none`, `python3 main.py`)
@LIA.r:       @LIA.eval(`["main.R"]`, `none`, `Rscript main.R`)
@LIA.rust:    @LIA.eval(`["main.rs"]`, `rustc main.rs`, `./main`)
@LIA.zig:     @LIA.eval(`["main.zig"]`, `zig build-exe ./main.zig -O ReleaseSmall`, `./main`)

@LIA.dotnet:  @LIA.dotnet_(@uid)

@LIA.dotnet_
<script>
var uid = "@0"
var files = []

files.push(["project.csproj", `<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>
</Project>`])

files.push(["Program.cs", `@input(0)`])

send.handle("input", (e) => {
    CodeRunner.send(uid, {stdin: e})
})
send.handle("stop",  (e) => {
    CodeRunner.send(uid, {stop: true})
});


CodeRunner.handle(uid, function (msg) {
    switch (msg.service) {
        case 'data': {
            if (msg.ok) {
                CodeRunner.send(uid, {compile: "dotnet build -nologo"})
            }
            else {
                send.lia("LIA: stop")
            }
            break;
        }
        case 'compile': {
            if (msg.ok) {
                if (msg.message) {
                    if (msg.problems.length)
                        console.warn(msg.message);
                    else
                        console.log(msg.message);
                }

                send.lia("LIA: terminal")
                console.clear()
                CodeRunner.send(uid, {exec: "dotnet run"})
            } else {
                send.lia(msg.message, msg.problems, false)
                send.lia("LIA: stop")
            }
            break;
        }
        case 'stdout': {
            if (msg.ok)
                console.stream(msg.data)
            else
                console.error(msg.data);
            break;
        }

        case 'stop': {
            if (msg.error) {
                console.error(msg.error);
            }

            if (msg.images) {
                for(let i = 0; i < msg.images.length; i++) {
                    console.html("<hr/>", msg.images[i].file)
                    console.html("<img title='" + msg.images[i].file + "' src='" + msg.images[i].data + "' onclick='window.LIA.img.click(\"" + msg.images[i].data + "\")'>")
                }

            }

            send.lia("LIA: stop")
            break;
        }

        default:
            console.log(msg)
            break;
    }
})


CodeRunner.send(
    uid, { "data": files }
);

"LIA: wait"
</script>
@end

@LIA.eval:  @LIA.eval_(false,@uid,`@0`,@1,@2)

@LIA.evalWithDebug: @LIA.eval_(true,@uid,`@0`,@1,@2)

@LIA.eval_
<script>
const uid = "@1"
var order = @2
var files = []

if (order[0])
  files.push([order[0], `@'input(0)`])
if (order[1])
  files.push([order[1], `@'input(1)`])
if (order[2])
  files.push([order[2], `@'input(2)`])
if (order[3])
  files.push([order[3], `@'input(3)`])
if (order[4])
  files.push([order[4], `@'input(4)`])
if (order[5])
  files.push([order[5], `@'input(5)`])
if (order[6])
  files.push([order[6], `@'input(6)`])
if (order[7])
  files.push([order[7], `@'input(7)`])
if (order[8])
  files.push([order[8], `@'input(8)`])
if (order[9])
  files.push([order[9], `@'input(9)`])


send.handle("input", (e) => {
    CodeRunner.send(uid, {stdin: e})
})
send.handle("stop",  (e) => {
    CodeRunner.send(uid, {stop: true})
});


CodeRunner.handle(uid, function (msg) {
    switch (msg.service) {
        case 'data': {
            if (msg.ok) {
                CodeRunner.send(uid, {compile: @3})
            }
            else {
                send.lia("LIA: stop")
            }
            break;
        }
        case 'compile': {
            if (msg.ok) {
                if (msg.message) {
                    if (msg.problems.length)
                        console.warn(msg.message);
                    else
                        console.log(msg.message);
                }

                send.lia("LIA: terminal")
                CodeRunner.send(uid, {exec: @4})

                if(!@0) {
                  console.clear()
                }
            } else {
                send.lia(msg.message, msg.problems, false)
                send.lia("LIA: stop")
            }
            break;
        }
        case 'stdout': {
            if (msg.ok)
                console.stream(msg.data)
            else
                console.error(msg.data);
            break;
        }

        case 'stop': {
            if (msg.error) {
                console.error(msg.error);
            }

            if (msg.images) {
                for(let i = 0; i < msg.images.length; i++) {
                    console.html("<hr/>", msg.images[i].file)
                    console.html("<img title='" + msg.images[i].file + "' src='" + msg.images[i].data + "' onclick='window.LIA.img.click(\"" + msg.images[i].data + "\")'>")
                }

            }

            send.lia("LIA: stop")
            break;
        }

        default:
            console.log(msg)
            break;
    }
})


CodeRunner.send(
    uid, { "data": files }
);

"LIA: wait"
</script>
@end



-->
<!--
nvm use v14.21.1
liascript-devserver --input README.md --port 3001 --live
link:     https://cdn.jsdelivr.net/gh/liascript/custom-style/custom.min.css
          https://cdn.jsdelivr.net/gh/andreainfufsm/elc106-2023a/classes/12/custom.css

-->


# Using LiaScript @UFSM Brazil

> First LiaScript User-Symposium, 6 Nov 2023<br><br>
> Prof. Andrea Schwertner Charão<br>
> Department of Languages and Computing Systems<br>
> Federal University of Santa Maria (UFSM), Brazil

 

<!-- --{{0}}-- I'm a professor at the Department of Languagens and Computing Systems at the Federal University of Santa Maria, Brazil. -->

<!-- --{{1}}-- UFSM is a large public higher education institution offering more than 200 graduate and undergraduate programs in nearly all knowledge areas, with almost 30,000 students enrolled -->


## We & LiaScript


                 {{1}}
************************************************

- Using Markdown and GitHub for many years 
- Discovered LiaScript on a GitHub Education community forum
- LiaScript users since Dec 2022


<!-- --{{1}}-- We have been using Markdown and GitHub for several years in our courses and projects. We discovered LiaScript on a forum and we have been using it since December 2022. -->


************************************************


                 {{2}}
************************************************

- Using LiaScript in

  - undergraduate courses in our Computer Science and Information Systems programs
  - internal reporting of research results

<!-- --{{2}}-- We mainly use LiaScript for our in-person classes of our Computer Science and Information Systems undergraduate programs. We have also been using LiaScript in research projects to gather results for internal presentation and reporting. -->



************************************************




## Favorite features 



                 {{1}}
************************************************

- The concept of rich web-based open courses built from Markdown
- One single document, multiple views (presentation, slides, textbook)


<!-- --{{1}}-- What we most like about LiaScript is the whole idea of building rich,  web-based open courses from Markdown-based content. Also, we very much appreciate the idea of having a single document with multiple possible view modes.-->



************************************************


                 {{2}}
************************************************

- Quizzes for formative assessment 


<!-- --{{2}}-- Quizzes are one of the most important LiaScript features for us. They are a powerful tool to build formative assessments. -->


************************************************


                 {{3}}
************************************************


- CodeRunner
- Classroom


<!-- --{{3}}-- We have also explored other features: CodeRunner and Classroom. We have used CodeRunner to present code snippets in Python, Java and Haskell. We have also used the Classroom chat feature, which is very engaging for students. -->



************************************************









## Custom tricks

We are always enthusiastic to implement some custom tricks... for example:

<!-- --{{0}}-- We are always enthusiastic to implement some custom tricks using LiaScript... for example.  -->


                 {{1}}
************************************************

CodeRunner on Repl.it
![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTvp3LgsE21kZVxveLmQb7qbhEhWaWSG1_xMDOPDlCkItzVpU40EITM_GPwrN6rMR_lW50&usqp=CAU)

<!-- --{{1}}-- We have managed to deploy CodeRunner servers on the Repl.it platform, for running Python and Java code demonstrations. This leverages the power of the free of this cloud-based platform and also the educational impact of public containers. -->

************************************************


                 {{2}}
************************************************

Integrated web form
![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTvp3LgsE21kZVxveLmQb7qbhEhWaWSG1_xMDOPDlCkItzVpU40EITM_GPwrN6rMR_lW50&usqp=CAU)

<!-- --{{1}}-- We have also used HTML and JavaScript to build web forms integrated with LiaScript, so students could provide structured data for some course activities.  -->

************************************************


                 {{3}}
************************************************

Animations
![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTvp3LgsE21kZVxveLmQb7qbhEhWaWSG1_xMDOPDlCkItzVpU40EITM_GPwrN6rMR_lW50&usqp=CAU)

<!-- --{{3}}-- Also, we have used CSS and JavaScript to implement some text animations.  -->

************************************************



## Challenges


Some challenges we face...

                 {{1}}
************************************************

Time to design a good quality interaction flow
![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTvp3LgsE21kZVxveLmQb7qbhEhWaWSG1_xMDOPDlCkItzVpU40EITM_GPwrN6rMR_lW50&usqp=CAU)

<!-- --{{1}}-- I think the greater challenge is that it takes a lot of time to design a good quality interaction flow and prepare content that fits for all 3 modes: presentation, slides and textbook.  -->

************************************************



                 {{2}}
************************************************

{{2}}
Visual aspects of presentations

- less impactful than "presentation-only" tools
- specially when layouting image compositions
- lots of blank areas in screen
- navigation usually requires left/right and vertical scrolling

![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTvp3LgsE21kZVxveLmQb7qbhEhWaWSG1_xMDOPDlCkItzVpU40EITM_GPwrN6rMR_lW50&usqp=CAU)


************************************************
    

## Wishlist


      {{1}}
************************************************

Navigation: more control over the side menu

- ability to hide some (continuing) titles
- ability to collapse

![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTvp3LgsE21kZVxveLmQb7qbhEhWaWSG1_xMDOPDlCkItzVpU40EITM_GPwrN6rMR_lW50&usqp=CAU)


************************************************


      {{2}}
************************************************

Community themes and templates

- custom impactful styles to impress our audience
- course "design patterns"

![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTvp3LgsE21kZVxveLmQb7qbhEhWaWSG1_xMDOPDlCkItzVpU40EITM_GPwrN6rMR_lW50&usqp=CAU)


************************************************
    




