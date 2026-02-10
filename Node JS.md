

**Introduction to NPM**

Node Package Manager, or NPM, is the default package manager for Node.js and is the world’s largest software registry. It consists of a command-line client (`npm`) and an online database of public and private packages, called the npm registry. NPM simplifies the process of sharing and reusing JavaScript code from other developers and libraries, which can significantly accelerate development.

**Managing Packages with NPM**

NPM makes it easy to install, update, and uninstall packages. Each Node.js project has a `package.json` file that manages the project's dependencies, scripts, and version info.

**Installing NPM and Setting Up** `**package.json**`

If you’ve installed Node.js, you already have NPM. To set up a new Node.js project with NPM:

1. **Create a new directory** for your project and navigate into it:

mkdir myproject  
cd myproject

**2. Initialize a new NPM project**:

npm init -y

This command creates a `package.json` file in your project directory with default values.

**Adding Dependencies**

To add a library or a framework to your project, you use the `npm install`command, specifying the package name. For example, to install Express.js:

npm install express

This command modifies the `package.json` file and adds the installed package to the dependencies list. It also creates a `node_modules` directory where all your installed packages and their dependencies are stored.

### 🚀 What `npm run` Does

The command **`npm run <script-name>`** is used to **run custom scripts defined in your project’s `package.json` file**. npm looks for a matching name under the `"scripts"` section and executes whatever command you’ve defined there. 

For example, if your `package.json` has:

`{   "scripts": {     "test": "echo \"Running tests...\"",     "start": "node index.js"   } }`

Then:

- `npm run test` will run `echo "Running tests..."`
    
- `npm run start` will run `node index.js`
    

> If you run just `npm run` without a script name, npm will list all scripts available. 

### 📦 Why Use `npm run`

- **Shortcuts for long commands:** Instead of typing a long build or test command manually, you define it once in `package.json` and just run it by name. 
- **Consistency:** Every developer runs the same commands. 
- **Automation:** You can chain tasks like building, testing, starting servers, etc.

## 📦 What Are _npm scripts_?

**npm scripts** are custom commands you define inside the `"scripts"` section of your project’s **`package.json`** file. They let you **run tasks (like starting a server, building code, running tests, etc.) with simple, memorable names instead of long shell commands**. 

---

## 🧠 Why They’re Useful

npm scripts help you:

- **Automate repetitive tasks** (e.g., build, lint, test) without typing long commands. 
    
- **Share commands with your team** — everyone runs tasks the same way. 
    
- **Integrate with other tools** like linters, bundlers, or test runners without installing them globally. 
    

---

## 📄 How You Define Scripts

In your `package.json`, you add a `"scripts"` section like this:

`{   "scripts": {     "start": "node app.js",     "build": "webpack --mode production",     "test": "jest",     "lint": "eslint ."   } }`

Here, `"start"`, `"build"`, `"test"`, and `"lint"` are script names you can run with npm. 

---

## ▶️ How to Run Scripts

Use **`npm run <script-name>`** to run any script:

`npm run build`

This will run the command you defined for `"build"`. 

There are **special shortcuts** for some common names:

- `npm start` runs the `start` script
    
- `npm test` runs the `test` script
    

For these, **you don’t need the `run` keyword**. 

---

## 🔁 Pre and Post Scripts

npm also supports **pre** and **post** hooks:

`{   "scripts": {     "prebuild": "echo preparing…",     "build": "webpack",     "postbuild": "echo done!"   } }`

- `prebuild` runs before `build`
    
- `postbuild` runs after `build`  
    This lets you chain commands in a predictable order. 
    

---

## 🚀 Examples of Common Tasks

Typical npm scripts you’ll see in projects:

|Script|Purpose|
|---|---|
|`start`|Start the app|
|`dev`|Run a development server|
|`build`|Compile or bundle code|
|`test`|Run automated tests|
|`lint`|Check code style|
|`clean`|Remove build artifacts|

These help you automate the workflow as your project grows. 

---

## 🧩 How npm Executes Scripts

When npm runs a script, it:

1. Looks up the name in the `"scripts"` section.
    
2. Executes the defined command in a shell.
    
3. Adds `node_modules/.bin` to the PATH, so local tools (like webpack, jest) can be used without installing them globally.


### 📌 What **`npm run`** means

The command **`npm run`** is used to **execute a named script from your project’s `package.json` file**. It runs whatever command you’ve defined in the `"scripts"` section under that name. 

---

### 🔹 How it works

When you run:

`npm run <script-name>`

npm looks in your project’s **`package.json`** under `"scripts"` for a script with that name, then **runs it in a shell** (like a terminal). 

For example, if your `package.json` includes:

`{   "scripts": {     "build": "webpack --mode production"   } }`

Then:

`npm run build`

executes the command behind `"build"` — here, running webpack in production mode. 

---

### 🔹 What `npm run` actually does

- It **starts a shell** and runs the defined command, just like typing the command manually. 
    
- It **adds `node_modules/.bin` to the PATH** so locally installed tools can be used without specifying full paths. 
    
- Scripts are run from the **root of the project folder**, no matter where you ran the command from. 
    

---

### 🧠 Special cases

- For _special_ script names like `start` or `test`, npm lets you omit the `run` keyword:
    
    `npm start npm test`
    
    These will run the `start` and `test` scripts respectively. 
    

---

### 📌 Summary

✔️ **`npm run <script>`** tells npm: “Run the script named `<script>` that’s defined in package.json.”   
✔️ Scripts run in your project’s root directory using a shell.   
✔️ You can automate any repeatable task (build, test, start server, etc.) this way.


