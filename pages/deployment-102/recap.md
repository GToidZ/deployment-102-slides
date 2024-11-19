---
layout: center
---

# Recap
What we have or haven't done last session

* Talked about concepts that associate with Deployment.
* Talked about PaaS and cloud components.
* Talked about common issues in going production in Django apps.
* Gone through the first objective in Activity 1.
  * We still have to work on `docker-compose.yaml` file.
  * Errata on the `Dockerfile`?

---

# Django `manage.py` shouldn't run in build step
I made a mistake.

Instead of treating built images from Docker as steps from setting up everything up until running a server,
it should be treated as an environment for the app to work on.

Let's consider:
* `migrate` - applies migrations that are not applied in database.
* `loaddata` - loads data fixtures into the database.
* `createsuperuser` - create a new superuser, affects database.

These shouldn't be run in build step, because database **does not** exist in the build step/environment.

---

# Django `manage.py` shouldn't run in build step
A way to fix,

We can instead delete all of the invocations of `manage.py` and put the important ones onto the last `CMD` statement.

`CMD` does not run in build step and is consider a command to run when you start a new container.

```diff
-RUN python ./manage.py migrate

-RUN python ./manage.py loaddata data

-RUN python ./manage.py createsuperuser --username admin --email admin@example.com --noinput

-CMD [ "python", "./manage.py", "runserver", "0.0.0.0:8000" ]
+CMD python ./manage.py migrate ; python ./manage.py runserver 0.0.0.0:8000
```

---

# Django `manage.py` shouldn't run in build step
A way to fix,

...or even better, you can have an entrypoint/script file that includes all commands you needed to be used when your container starts.

### `Dockerfile`
```diff
+COPY entrypoint.sh .
+RUN chmod +x ./entrypoint.sh

-CMD python ./manage.py migrate ; python ./manage.py runserver 0.0.0.0:8000
+CMD [ "./entrypoint.sh" ]
```

<br>

### `entrypoint.sh`
```sh
#!/bin/sh
python ./manage.py migrate
python ./manage.py runserver 0.0.0.0:8000
```

---

# Outline
Let's plan out how we can do so,

* [Make a working `Dockerfile` for `ku-polls`.]{style="color: #bbb;"}
    * [Browse a suitable image in Docker Hub.]{style="color: #bbb;"}
    * [Omit the unwanted files with `.dockerignore`.]{style="color: #bbb;"}
    * [Replicate steps to build our application with `Dockerfile`.]{style="color: #bbb;"}
* Create a `docker-compose.yaml` with `ku-polls` and `PostgreSQL`.
    * Write a service for database.
        * We'll use the image, `postgres`.
    * Write a service for our application.
* [et voilà!]{style="background: linear-gradient(to right, #ef5350, #f48fb1, #7e57c2, #2196f3, #26c6da, #43a047, #eeff41, #f9a825, #ff5722); -webkit-background-clip: text; -webkit-text-fill-color: transparent;"}

---