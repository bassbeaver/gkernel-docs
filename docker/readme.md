We are using MkDocs for document generation.

Explanation on how to use dockerized mkdocs:
```bash
docker run --rm -it -p 8000:8000 -v ${PWD}:/docs squidfunk/mkdocs-material --help
```

Run serving as a site
```bash
 docker run --rm -it -p 8000:8000 -v ${PWD}:/docs squidfunk/mkdocs-material
```

Deploy code to GitHub Pages
```bash
docker run --rm -it -p 8000:8000 -v ${PWD}:/docs squidfunk/mkdocs-material gh-deploy
```
