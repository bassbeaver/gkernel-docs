We are using MkDocs for document generation.

Explanation on how to use dockerized mkdocs:
```bash
docker run -i --rm melopt/mkdocs
```

Shell into container with docs mounted as volume:
```bash
docker run -i --rm --volume=$(pwd):/docs -p 8000:8000 melopt/mkdocs sh
```
