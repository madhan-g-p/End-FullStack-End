# NodeJS Installation and Setup

## Overview

- Node.js is an open-source runtime environment that allows you to run JavaScript code outside of a web browser. 
- It enables developers to build fast, scalable network applications and server-side scripts. 
- It is made up of Google Chrome Browser's V8 Javascript engine 
- The Javascript engine is wrapped over several C++ libs like libuv and others to perform async network and I/O operations

## Core Use Cases
**Web APIs & Backends**: It is most commonly used to build the "brain" of websites—the server-side logic that interacts with databases, handles user authentication, and processes requests.

**Real-Time Applications**: Because it handles many simultaneous connections with very low delay (latency), it is the go-to choice for Chat Apps, online gaming servers, and live collaboration tools like Trello or Slack.

**Data Streaming**: Node.js can process data while it is still being uploaded, making it highly efficient for services like Netflix or YouTube that stream massive amounts of video and audio data without buffering the entire file first.

**Microservices**: Its lightweight nature makes it perfect for breaking a large, complex application into smaller, independent services that are easier to manage and scale.

**Internet of Things (IoT)**: It can efficiently manage requests from thousands of connected devices (like smart sensors or home appliances) simultaneously.

**Command-Line Tools (CLI)**: Many developers use Node.js to create automation scripts and tools for their own development workflows. 

## Key Benefits
**JavaScript Everywhere**: You can use the same programming language for both the frontend (browser) and backend (server), allowing for easier code sharing and smaller, more versatile development teams.

**High Performance**: It uses an "asynchronous, non-blocking" model. Instead of waiting for one task to finish (like reading a file) before starting the next, it keeps working on other tasks in the background, making it incredibly fast for heavy input/output (I/O) work.

**Massive Ecosystem**: Through the NPM Registry, developers have access to millions of free, ready-to-use code packages, which significantly speeds up the building process. 

### When NOT to use it
Node.js is generally **not recommended for CPU-intensive tasks, such as complex mathematical calculations, heavy video encoding, or deep data analysis**. Because it is single-threaded, these heavy tasks can "block" the server, making it unresponsive to other users until the calculation is done

## Installation
 Just like any other software NodeJs releases comes as versionzed , each version having it's own bug fixes and patch fixes. So a project 's node engine and package must always stay in sync. The better way to install and manage node versions is using the NVM (_The Node Version Manager_).
  - [Installation for Windows Users](https://github.com/coreybutler/nvm-windows)
  - [Installation for Mac-OS or Linux Users](https://github.com/nvm-sh/nvm)

## Package Manager
  Ofcourse NodeJS Applications can be built with pre built library packages from the popular package registry [npmjs](http://npmjs.com/)
 
  The [`pnpm`](https://pnpm.io/installation) and [`npm`](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) are most popular package managers to download and install packages in your project. Though I personally prefer `pnpm`, I am leaving the comparison table below for you to choose one between them.

### Comparison between `npm` and `pnpm`
  | Feature                                | npm                                                | pnpm                                                                 |
| -------------------------------------- | -------------------------------------------------- | -------------------------------------------------------------------- |
| **Package Storage**                    | Stores a separate copy of dependencies per project | Uses a **global content-addressable store** with hard links/symlinks |
| **Disk Space Usage**                   | Higher disk usage                                  | Much lower disk usage                                                |
| **Install Speed**                      | Fast, but duplicates packages                      | Usually faster after first install due to shared cache               |
| **node_modules Structure**             | Flat hoisted structure                             | Strict, symlinked structure                                          |
| **Dependency Resolution**              | Can accidentally access undeclared deps            | Prevents phantom dependencies                                        |
| **Monorepo Support**                   | Basic workspaces support                           | Excellent native workspace support                                   |
| **Deterministic Installs**             | Good with `package-lock.json`                      | Very strict and reproducible with `pnpm-lock.yaml`                   |
| **Security**                           | Standard dependency isolation                      | Better isolation between packages                                    |
| **Compatibility**                      | Maximum ecosystem compatibility                    | Mostly compatible, occasional edge-case issues                       |
| **CI/CD Performance**                  | Good                                               | Often faster and more cache-efficient                                |
| **Default Package Manager in Node.js** | Yes                                                | No                                                                   |
| **Learning Curve**                     | Minimal                                            | Slightly higher due to symlink behavior                              |
| **Command Example**                    | `npm install lodash`                               | `pnpm add lodash`                                                    |
| **Best For**                           | Simplicity, broad compatibility                    | Large apps, monorepos, performance optimization                      |

#### Quick Take
| Use Case                 | npm  | pnpm  |
| ------------------------ | :---: | :----: |
| Small/simple projects    | ⭐⭐⭐⭐⭐ |   ⭐⭐⭐  |
| Large monorepos          |  ⭐⭐⭐  |  ⭐⭐⭐⭐⭐ |
| Saving disk space        |   ⭐⭐  |  ⭐⭐⭐⭐⭐ |
| Maximum compatibility    | ⭐⭐⭐⭐⭐ |  ⭐⭐⭐⭐  |
| Faster repeated installs |  ⭐⭐⭐  |  ⭐⭐⭐⭐⭐ |
| Teams with many packages |  ⭐⭐⭐  |  ⭐⭐⭐⭐⭐ |

#### Example analogy
| Scenario               | npm                      | pnpm                 |
| ---------------------- | ------------------------ | -------------------- |
| 5 projects using React | ~5 separate React copies | 1 shared React store |
| Disk impact            | High                     | Low                  |

#### Frequent Commands
| Action               | npm                     | pnpm                  |
| -------------------- | ----------------------- | --------------------- |
| Install dependencies | `npm install`           | `pnpm install`        |
| Add package          | `npm install express`   | `pnpm add express`    |
| Remove package       | `npm uninstall express` | `pnpm remove express` |
| Run script           | `npm run dev`           | `pnpm dev`            |
| Update deps          | `npm update`            | `pnpm update`         |

#### Recommendation
- Choose npm if you want:
    - zero friction
    - widest compatibility
    - standard Node.js defaults
- Choose pnpm if you want:
    - faster installs
    - lower disk usage
    - cleaner dependency management
    - strong monorepo tooling

