
## Step-by-Step Instructions to Implement the Workflow
[Documentation](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/running-variations-of-jobs-in-a-workflow)
1. **Create the Workflow File**  
   Go to the `.github/workflows` directory in your repository and create a new YAML file for the workflow.

2. **Set the Workflow Name and Trigger**  
   At the top of the file, specify a name for the workflow (e.g., `Matrix Lab`) and set it to be triggered manually using the `workflow_dispatch` event.

3. **Define the Job and Matrix**  
   Add a job (for example, `example_matrix`).  
   Under the job, define a matrix with two operating systems (`windows-latest` and `ubuntu-latest`) and two Node.js versions (`14` and `16`).

4. **Configure the Runner**  
   Set the job to run on the operating system specified by the matrix for each combination.

5. **Add Steps to the Job**  
   - Use the Node.js setup action to install the Node.js version specified by the matrix.

6. **Save and Commit**  
   Save the workflow file and commit it to your repository.

7. **Run the Workflow**  
   Trigger the workflow manually from the GitHub Actions tab and review the results for each matrix combination.

<details>
  <summary>Solution</summary>

```YAML
name: Matrix Lab
on:
  workflow_dispatch:
jobs:
  example_matrix:
    strategy:
      matrix:
        os: [windows-latest, ubuntu-latest]
        node: [14, 16]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
```

</details>


1. **Add an Extra Matrix Combination**  
   Use the `include` keyword in the matrix to add an additional variable (`npm: 6`) for the combination where the OS is `windows-latest` and Node.js version is `16`.

2. **Add Steps to the Job**  
   - Add a conditional step to install a specific npm version (`npm@6`) only if the `npm` variable is present in the matrix.
   - Add a step to print the npm version.

<details>
  <summary>Solution</summary>

```YAML
name: Matrix Lab
on:
  workflow_dispatch:
jobs:
  example_matrix:
    strategy:
      matrix:
        os: [windows-latest, ubuntu-latest]
        node: [14, 16]
        include:
          - os: windows-latest
            node: 16
            npm: 6
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - if: ${{ matrix.npm }}
        run: npm install -g npm@${{ matrix.npm }}
      - run: npm --version
```

</details>

## Optional 
**Add a Maximum Parallel Jobs Setting**  
In the matrix strategy for your job, add the `max-parallel: 2` option. This setting ensures that no more than two matrix job combinations run at the same time, which can help manage resource usage or avoid hitting concurrency limits.

<details>
  <summary>Solution</summary>

```YAML
name: Matrix Lab
on:
  workflow_dispatch:
jobs:
  example_matrix:
    strategy:
      max-parallel: 2
      matrix:
        os: [windows-latest, ubuntu-latest]
        node: [14, 16]
        include:
          - os: windows-latest
            node: 16
            npm: 6
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - if: ${{ matrix.npm }}
        run: npm install -g npm@${{ matrix.npm }}
      - run: npm --version
```

</details>

**Make this workflow reusable between repositories**

Remove the `workflow_dispatch` trigger and add the `workflow_call` trigger and commit the changes. If your repository is not `public`, then In the repository that resides your now reusable workflow, go to the repository settings -> actions -> general and check the box `Accessible from repositories owned by the user ....`. Create a workflow that consumes this workflow in a new repository by calling the reusable workflow `organzation/repository/.github/workflows/matrix-lab.yml@main`

<details>
  <summary>Solution</summary>

```YAML
name: Matrix Lab
on:
  workflow_call:
jobs:
  example_matrix:
    strategy:
      max-parallel: 2
      matrix:
        os: [windows-latest, ubuntu-latest]
        node: [14, 16]
        include:
          - os: windows-latest
            node: 16
            npm: 6
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - if: ${{ matrix.npm }}
        run: npm install -g npm@${{ matrix.npm }}
      - run: npm --version
```

</details>

<details>
  <summary>Solution</summary>

```YAML
name: Consume workflow
on:
  workflow_dispatch:
jobs:
  consume_workflow:
    - uses: organzation/repository/.github/workflows/matrix-lab.yml@main
```

</details>
