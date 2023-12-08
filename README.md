## Hello 👋! There you will learn how to deploy the project with nginx

### Firstly we will create folders where our projects will be stored
```
mkdir var
```

```
cd /var
```

```
mkdir wwww
```

```
cd www
```

### And now we will clone our project to server

#### Download git

```
apt install git
```

#### Clone the project

```
git clone git@github.com:username/project_name.git
```

#### Enter to the project

```
cd project_name
```

### And let's install packages

#### Install virtual environment to project
```
pip install virtualenv
```

#### Install all packages from requirements.txt

```
pip install -r requirements.txt
```
