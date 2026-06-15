# How to run sprintstart-backend

## Prerequisites

- Git
- JVM 21

## Cloning the repo

```sh
$ git clone https://github.com/SprintStartProject/sprintstart-backend
```

## Navigate to the desired branch

Switch to the branch of the version you want to run:

- `main`: Contains changes from completed sprints only
- `dev`: Latest finished changes
- `feature/`: The banches for the actual features, still in development

## Building the application

_For running the application not necessary, but recommended to see if the current state is stable_

From the root of the project run:

```sh
$ ./gradlew build
```

## Running the application

To run the application, you'll have to meet some additional prerequisites:

### DB

You have to have a local postgres server running. This server must have a separate db for this project.
Also, you'll have to add a `run.local.env` file in the project root, following the standards of the `run.local.env.example` file.

```env
# Local Spring Boot backend run config
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:15432/sprintstart
SPRING_DATASOURCE_USERNAME=sprintstart
SPRING_DATASOURCE_PASSWORD=change-me
```

### Running the application

Once these prerequisites are met, you can run the application by running:

```bash
./gradlew bootRun
```

from the project root.
