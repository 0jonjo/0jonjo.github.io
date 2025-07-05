---
layout: post
title: "Updating Calcpace with Gemini CLI"
description: "Exploring Google's Generative AI Command Line Interface"
tags: programming english ruby
categories: misc
youtubeId:
image:
---

A few weeks ago, the first public releases of [Gemini CLI](https://github.com/google-gemini/gemini-cli) were launched. I was curious to try out Google's solution for generative AI in the command line, similar to Claude Code or OpenAI CLI. I wanted to see it in action with a real project, so I decided to work on the next update of my gem [Calcpace](https://rubygems.org/gems/calcpace) with it. It's a project about calculations and conversions of distance, time and velocity that has more than 3,400 downloads on RubyGems.

Installing Gemini CLI in Ubuntu is straightforward - just follow the instructions in the [README](https://github.com/google-gemini/gemini-cli#quickstart). After that, you can use the `gemini` command to interact with the Gemini AI model. When using it for the first time, you need to authenticate with your Google account and set some simple configurations like theme. Currently, the CLI is free, but the model utilized is the powerful Gemini 2.5 Pro, which is the same one used in other Google paid products.

I did all work inside my Calcpace project directory, and my initial good impression was that it automatically and quickly read all the project files and directories. As suggested in the documentation, I asked the AI to summarize the project, and it returned the main features, version, and license of the gem in a short and accurate summary. After that, I asked for a more detailed analysis of the codebase, and it provided a good overview of the main classes and methods. Then I requested it to identify potential improvements in the code, and it suggested some refactoring opportunities, such as quick wins to improve the project. Gemini CLI suggested improving the error handling that was already on my to-do list, and it also pointed out handling edge cases that I hadn't considered yet, like user input in "XX:XX" format, which is not currently supported by the gem (only "XX:XX:XX" format). This was a great insight, as I hadn't thought about this case before.

Instead of just following the instructions, I prompted it to write a plan - in a Markdown file - on how to implement these improvements, and it provided a clear and concise plan with steps to follow. Then I asked it to follow the instructions of each part of the plan. Whenever Gemini CLI has to make a change in a file or run a command in the terminal, it asks for confirmation before proceeding, which is a good safety feature. I found this very useful, as it prevents accidental changes to the codebase and allows you to review the changes both before and after they are made.

Another interesting aspect is that when Gemini CLI experiences some delay in answering or executing a command, it automatically switches the model to Gemini 2.5 Flash, which is faster to respond and execute commands, though with a slightly lower quality. In general, the quality of changes and suggestions made by Gemini CLI is very good. I also used it to make minor changes in my pipeline, update the [README](https://github.com/0jonjo/calcpace), and modify the file with specifications of the gem to upload to RubyGems. The AI was able to understand the context of the project and provide relevant suggestions and changes.

This is the Pull Request I created with the changes made using the Gemini CLI: [[57] Improvements in calculation conversion and how handle errors](https://github.com/0jonjo/calcpace/pull/58/files)

Here is a summary of the main features of Gemini CLI that I found useful:

- **Project Analysis**: It quickly reads and summarizes the project files, providing an overview of the main features, version, and license of the gem.
- **Code Review**: It identifies potential improvements in the code, suggesting refactoring opportunities and enhancements to error handling.
- **Plan Generation**: It can create a clear and concise plan for implementing improvements, breaking down the steps needed to achieve the desired changes.
- **Confirmation Prompts**: Before making any changes, it asks for confirmation, allowing you to review the changes before they are applied.
- **Model Switching**: It automatically switches to a faster model (Gemini 2.5 Flash) when there are delays in responses or command execution, ensuring a smoother experience.

One aspect I particularly do not like in Gemini CLI is that you cannot see it using your existing terminal, or suggest it to use your terminal to run commands like tests or start the server. You cannot directly interact with the terminal like you can with Copilot, which allows you to write code directly in the editor.

In general: ~~sorry Sarah Connor, Gemini CLI is my best friend now.~~ I found Gemini CLI faster and more efficient than Github Copilot with Claude Sonnet 3.7 or 4.0. After this initial experience, I started to use both tools together: using Gemini CLI for project analysis and planning, and then leveraging Copilot for specific code changes, tests, and enhancements. This is my current strategy to learn more about both tools and how they can complement each other.

Image: Meme of Sarah Connor (Linda Hamilton), from [The Exterminator](https://en.wikipedia.org/wiki/The_Terminator).
