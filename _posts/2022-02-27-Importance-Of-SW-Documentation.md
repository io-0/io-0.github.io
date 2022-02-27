# Importance Of Software Documentation
## Prelude
In my early stages as a software developer, I wrote code first and documented later. This made documenting a cumbersome task that I had to force myself to do after I had done what I enjoyed.
When I ran out of time I tend to skip it altogether.

Oh boy, I learned to regret that fast. Software gets outdated and insecure, requirements not so much. It happened to me more than once that I got the job to rewrite software. Sometimes migration to newer framework versions and technologies was not feasible. Other times, I was asked to switch the stack and language altogether for e.g. better performance or hosting capabilities. If no one documented what that software was capable of, you end up analysing for ages (and in my case also cursing a lot because the code war not written to be read by humans).
An other example where the lack of documentation made my job very hard was debugging of running systems. If no one cared to do proper logging, all you can do is try to redo what the user tells you to and pray that the bug/error happens again. If it doesn't, you are screwed.

Nowadays I mix documentation with coding and I am far better of. Time and budget constraints still haunt me daily so I also defined a order of importance to the different ways to document software. The lower it gets, the easier it is to do it later if one was forced to skip it.

## Documenting Use-cases And Requirements
This is the most important documentation for me. I want to be able to state what the software I wrote is capable of and what it is designed to do.
In all my software projects I work with [GIT](https://git-scm.com/) and [Jira](https://www.atlassian.com/software/jira) or a tool like it where use-cases and features are discussed and/or documented prior to implementation. It helps to reference in each commit what it relates to in Jira. But there is always that lingering fear that referenced (Jira) sources vanish for whatever reason. And sometimes it is cumbersome to follow references.
I switched to test driven development some time ago and that learned me a better way to document use-cases. For each use-case I start with a test that documents it. In this test I write comments about the use-case and tread the software as blackbox. Since I produce [Spring Boot](https://spring.io/projects/spring-boot) based micro-services with REST-APIs most of the time nowadays, these tests consist mostly of the following: A [webclient](https://spring.getdocs.org/en-US/spring-framework-docs/docs/testing/integration-testing/webtestclient.html) to call the API on a running instance. One or more pre recorded responses (thx [WireMock](https://wiremock.org/)) if the service depends on others. A temporary test DB, and finally request and response fixtures (mostly [YAML](https://yaml.org/) files).
This works great. No mocking hinders me from re-factoring! And if anyone needs to know what this micro-service can do, I point them to the tests.

## Documenting APIs
This is my second most important documentation. Let other developers know how they can interact with the software. As I stated earlier, I mostly write micro-services nowadays that provide REST-APIs. I found [OAS](https://swagger.io/specification/) to be a very good way to document that. There are tools that do a great job visualising the Specs and I can use code generation for models and request/response validation. I also tend to do API first software development. This helps greatly with team decoupling.

## Documenting What Happens In The Code
The next on my agenda are my team members and the future me. Software evolves all the time and is read way more often than written. Documentation here is done with proper layers/structure and good variable, method, function, interface and class names. After I turned my test green and the software can handle the new use-case I re-factor with the code reader in mind. I have simple rules: Forcing a reader to scroll to the right is impolite. The code should be structured in a way that it is easy to navigate through and understand. Like in a book the start gives an overview and the deeper you get the more details and inner workings are presented to the reader. I try to get at least one prove reader (code reviewer) because it happens that something that seams clear to you isn't for someone else. Code that is not straight forward because of business logic or workarounds gets a comment stating that. I try to avoid other comments (even Javadoc) because they tend to rot fast.

## Documenting What Happens At Runtime
Proper logging and metrics can save your a lot of stress and are sometimes the only way to understand what happens. They also point you to overlooked use-cases and performance issues. Always think about the proper log level tough. I can't count any more how often I saw software struggle or die because of filled (virtual) hard drives with logs. Modern systems tend to aggregate and index logs where too much unnecessary logging is a problem as well.

## Documenting Context, History and Usage
This is where Readmes and Changelogs come in handy. Because they are close to the code they have a higher chance to stay up to date. With a well maintained Changelog it is easy to understand what happened to the software over time and why it is in its current state. It should be easer to read than a commit history.
Readmes should at least give an overview what the software is about and its context. It should also contain how to use the software and what configuration options exist.

## Documenting Architecture and the Bigger Picture
Well, this is quite important, but not that important for me if I have the developer role because I have architects that do that for me. If I have the architecture role (as well) this changes of course and becomes the first thing I think about and document.

## User Manuals And Public API Documentation
I put this last because for me it seems the easiest to do at a later stage if the prior documentation was done. Internal API documentation should exists already and writing a public one should only be a matter of copying parts of the internal one and maybe redact it a little. Regarding user manuals I got lucky. I only had to do them once or twice and had no problem writing them afterwards.