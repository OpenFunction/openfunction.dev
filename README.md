# OpenFunction Website

This project uses [Hugo](https://gohugo.io/) and [Hugo template Docsy](https://github.com/google/docsy) to build the OpenFunction website.

## Contribute

Contributions of any kind are welcome!

### Fork and clone the repository

1. Fork the repository.

2. Run the following commands to clone your fork and enter into it. Make sure you replace `<Your GitHub ID>` with your GitHub ID.

   ```
   git clone https://github.com/<Your GitHub ID>/openfunction.dev.git
   cd openfunction.dev
   ```
### Build and preview the openfunction.dev

#### Pre-requisites

- [Hugo extended version](https://gohugo.io/getting-started/installing)
- [Node.js](https://nodejs.org/en/)

#### Environment setup

1. Ensure pre-requisites are installed
1. Clone repository
1. Change to openfunction.dev directory: `cd openfunction.dev`
1. Add Docsy submodule: `git submodule add https://github.com/google/docsy.git themes/docsy`
1. Update submodules: `git submodule update --init --recursive`
1. Install npm packages: `npm install`

### Running the openfunction.dev locally

After you setup the environment, run the following command:

```
hugo server -D
```

Now you can preview the openfunction.dev in your browser at `http://localhost:1313/`.

### Open a pull request

Open a [pull request (PR)](https://help.github.com/en/desktop/contributing-to-projects/creating-an-issue-or-pull-request#creating-a-new-pull-request) to contribute to our openfunction.dev. Use DCO sign-off when you submit a PR by running the command below (add `-s`):

```bash
git commit -s -m "xxx"
```