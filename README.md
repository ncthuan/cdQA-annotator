# cdQA-annotator

![GitHub](https://img.shields.io/github/license/cdqa-suite/cdQA-annotator.svg)

A web-based annotator for closed-domain question answering datasets with SQuAD format

![](https://cdqa-suite.github.io/cdQA-website/img/suite-3.5c84e524.png)

# Installation:

```
# Download
git clone https://github.com/cdqa-suite/cdQA-annotator
cd cdQA-annotator
```

## npm
```shell
# Install dependencies
npm install

# Start development server
npm run serve
```

## Docker
```
docker build . -t cdqa-annotator
docker run cdqa-annotator -p 8080:8080
```

The app should be running at http://localhost:8080/