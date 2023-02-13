# Hello world javascript action

This action prints "Hello World" or "Hello" + the name of a person to greet to the log.

## My notes

### Node.js and npm

Here we use a JavaScript action. The action will use Node.js 16.x as the language
to do the action. When using Node.js you also use Node Package Manager (npm) to download and
add packages to your project. Once the project directory is made (hello-world-javascript-action),
we initialise npm using the following

```
npm init -y
```

We will use the `@actions` packages on npm to help us along the way

```
npm install @actions/core
npm install @actions/github
```

Now you should see a `node_modules` directory with the modules you just installed and 
a `package-lock.json` file with the installed module dependencies and the versions of each installed module.

### The action metadata file

The action metadata describes the action, and includes such things as the `name`, `description`, `inputs`,
`output` and what it `runs`. These are essentially config to functions e.g.

```
# an 'input' function
who-to-greet(id, default = 'World', required = TRUE)
```

```
# action.yaml
name: 'Hello World'
description: 'Greet someone and record the time'
inputs:
  who-to-greet:  # id of input
    description: 'Who to greet'
    required: true
    default: 'World'
outputs:
  time: # id of output
    description: 'The time we greeted you'
runs:
  using: 'node16'
  main: 'index.js'
```

### The action code

Here, this is locked in `index.js`. We can see how the `@actions` packages are used as helpers to do things.

```
const core = require('@actions/core');
const github = require('@actions/github');

try {
  // `who-to-greet` input defined in action metadata file
  const nameToGreet = core.getInput('who-to-greet');
  console.log(`Hello ${nameToGreet}!`);
  const time = (new Date()).toTimeString();
  core.setOutput("time", time);
  // Get the JSON webhook payload for the event that triggered the workflow
  const payload = JSON.stringify(github.context.payload, undefined, 2)
  console.log(`The event payload: ${payload}`);
} catch (error) {
  core.setFailed(error.message);
}
```

### Commit and tagging

We commit everything, and also `git tag` for versioning
```
git add action.yml index.js node_modules/* package.json package-lock.json README.md
git commit -m "My first action is ready"
git tag -a -m "My first action release" v1.1
git push --follow-tags
```

### Externally using the action

This would be used by an **external** repo. Note the use of `@v1.1` to help with versioning of the action
```
on: [push]

jobs:
  hello_world_job:
    runs-on: ubuntu-latest
    name: A job to say hello
    steps:
      - name: Hello world action step
        id: hello
        uses: shaunson26/hello-world-javascript-action@v1.1
        with:
          who-to-greet: 'Mona the Octocat'
      # Use the output from the `hello` step
      - name: Get the output time
        run: echo "The time was ${{ steps.hello.outputs.time }}"
```

### Interally using the action

```
on: [push]

jobs:
  hello_world_job:
    runs-on: ubuntu-latest
    name: A job to say hello
    steps:
      # To use this repository's private action,
      # you must check out the repository
      - name: Checkout
        uses: actions/checkout@v3
      - name: Hello world action step
        uses: ./ # Uses an action in the root directory
        id: hello
        with:
          who-to-greet: 'Mona the Octocat'
      # Use the output from the `hello` step
      - name: Get the output time
        run: echo "The time was ${{ steps.hello.outputs.time }}"
```


## Inputs

### `who-to-greet`

**Required** The name of the person to greet. Default `"World"`.

## Outputs

### `time`

The time we greeted you.

## Example usage

```yaml
uses: shaunson26/hello-world-javascript-action@v1.1
with:
  who-to-greet: 'Mona the Octocat'
```