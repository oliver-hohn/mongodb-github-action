<div align="center">
  <a href="https://superchargejs.com">
    <img width="471" style="max-width:100%;" src="https://superchargejs.com/images/supercharge-text.svg" />
  </a>
  <br/>
  <br/>
  <p>
    <h3>MongoDB in GitHub Actions</h3>
  </p>
  <p>
    Start a MongoDB server in your GitHub Actions.
  </p>
  <br/>
  <p>
    <a href="#usage"><strong>Usage</strong></a>
  </p>
  <br/>
  <br/>
  <p>
    <em>Follow <a href="http://twitter.com/marcuspoehls">@marcuspoehls</a> and <a href="http://twitter.com/superchargejs">@superchargejs</a> for updates!</em>
  </p>
</div>

---


## Introduction
This GitHub Action starts a MongoDB server or MongoDB replica set. By default, the MongoDB server is available on the default port `27017`. You can configure a custom port using the `mongodb-port` input. The examples show how to use a custom port.

The MongoDB version must be specified using the `mongodb-version` input. The used version must exist in the published [`mongo` Docker hub tags](https://hub.docker.com/_/mongo?tab=tags). Default value is `latest`, other popular choices are `6.0`, `7.0` or even release candidates `8.0.0-rc4`.

This is useful when running tests against a MongoDB database.


## Usage
A code example says more than a 1000 words. Here’s an exemplary GitHub Action using a MongoDB server in different versions to test a Node.js app:

```yaml
name: Run tests

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [20.x, 22.x]
        mongodb-version: ['6.0', '7.0', '8.0']

    steps:
    - name: Git checkout
      uses: actions/checkout@v4

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}

    - name: Start MongoDB
      uses: supercharge/mongodb-github-action@1.11.0
      with:
        mongodb-version: ${{ matrix.mongodb-version }}

    - run: npm install

    - run: npm test
      env:
        CI: true
```

### Logging into Dockerhub
By default, this Github action does not authenticate requests to pull public images from Dockerhub. This means that in some cases the pull request is rate limited by Dockerhub: https://docs.docker.com/docker-hub/download-rate-limit/#how-do-i-authenticate-pull-requests (unauthorized requests are rate limited by IP!).

To overcome this, you can log into Dockerhub using the `dockerhub-username` and `dockerhub-password` inputs.
- `dockerhub-username`: the "docker ID" (aka username) used to log into the desired Dockerhub account.
- `dockerhub-password`: the password for the desired Dockerhub account or a personal access token for the account. _Best practices is to use a personal access token, so that access can be scoped down: https://docs.docker.com/security/for-developers/access-tokens/_

```yaml
name: Run tests

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [20.x, 22.x]
        mongodb-version: ['6.0', '7.0', '8.0']

    steps:
    - name: Git checkout
      uses: actions/checkout@v4

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}

    - name: Start MongoDB
      uses: supercharge/mongodb-github-action@1.11.0
      with:
        # To avoid leaking your credentials in plaintext, use Github secrets to hold the Dockerhub credentials
        dockerhub-username: ${{ secrets.DOCKERHUB_USERNAME }}
        dockerhub-password: ${{ secrets.DOCKERHUB_PASSWORD }}
        mongodb-version: ${{ matrix.mongodb-version }}

    - run: npm install

    - run: npm test
      env:
        CI: true
```

### Using a Custom MongoDB Port
You can start the MongoDB instance on a custom port. Use the `mongodb-port: 12345` input to configure port `12345` for MongoDB. Replace `12345` with the port you want to use in your test runs.

The following example starts a MongoDB server on port `42069`:

```yaml
name: Run tests

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [20.x, 22.x]
        mongodb-version: ['6.0', '7.0', '8.0']

    steps:
    - name: Git checkout
      uses: actions/checkout@v4

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}

    - name: Start MongoDB
      uses: supercharge/mongodb-github-action@1.11.0
      with:
        mongodb-version: ${{ matrix.mongodb-version }}
        mongodb-replica-set: test-rs
        mongodb-port: 42069

    - name: Install dependencies
      run: npm install

    - name: Run tests
      run: npm test
      env:
        CI: true
```


### With a Replica Set (MongoDB `--replSet` Flag)
You can run your tests against a MongoDB replica set by adding the `mongodb-replica-set: your-replicate-set-name` input in your action’s workflow. The value for `mongodb-replica-set` defines the name of your replica set. Replace `your-replicate-set-name` with the replica set name you want to use in your tests.

The following example uses the replica set name `test-rs`:

```yaml
name: Run tests

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [20.x, 22.x]
        mongodb-version: ['6.0', '7.0', '8.0']

    steps:
    - name: Git checkout
      uses: actions/checkout@v4

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}

    - name: Start MongoDB
      uses: supercharge/mongodb-github-action@1.11.0
      with:
        mongodb-version: ${{ matrix.mongodb-version }}
        mongodb-replica-set: test-rs
        mongodb-port: 42069

    - name: Install dependencies
      run: npm install

    - name: Run tests
      run: npm test
      env:
        CI: true
```


### With Authentication (MongoDB `--auth` Flag)
Setting the `mongodb-username` and `mongodb-password` inputs. As per the [Dockerhub documentation](https://hub.docker.com/_/mongo), this automatically creates an admin user and enables `--auth` mode.

The following example uses the username `supercharge`, password `secret` and also sets an initial `supercharge` database:

```yaml
name: Run tests with authentication

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [20.x, 22.x]
        mongodb-version: ['6.0', '7.0', '8.0']

    steps:
    - name: Git checkout
      uses: actions/checkout@v4

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}

    - name: Start MongoDB
      uses: supercharge/mongodb-github-action@1.11.0
      with:
        mongodb-version: ${{ matrix.mongodb-version }}
        mongodb-username: supercharge
        mongodb-password: secret
        mongodb-db: supercharge

    - name: Install dependencies
      run: npm install

    - name: Run tests
      run: npm test
      env:
        CI: true
```

### With Custom Container Name
The container name of the created MongoDB instance can be configured using the `mongodb-container-name` input

The following example will parameterize the MongoDB container name based on the `node-version` and `mongodb-version` being used from the matrix:

```yaml
name: Run with Custom Container Names

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [20.x, 22.x]
        mongodb-version: ['6.0', '7.0', '8.0']

    steps:
    - name: Git checkout
      uses: actions/checkout@v4

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}

    - name: Start MongoDB
      uses: supercharge/mongodb-github-action@1.11.0
      with:
        mongodb-version: ${{ matrix.mongodb-version }}
        mongodb-container-name: mongodb-${{ matrix.node-version }}-${{ matrix.mongodb-version }}

    - name: Install dependencies
      run: npm install

    - name: Run tests
      run: npm test
      env:
        CI: true
```

**Caveat:** due to [this issue](https://github.com/docker-library/mongo/issues/211), you **cannot enable user creation AND replica sets** initially. Therefore, if you use this action to setup a replica set, please create your users through a separate script.

## License
MIT © [Supercharge](https://superchargejs.com)

---

> [superchargejs.com](https://superchargejs.com) &nbsp;&middot;&nbsp;
> GitHub [@supercharge](https://github.com/supercharge) &nbsp;&middot;&nbsp;
> Twitter [@superchargejs](https://twitter.com/superchargejs)
