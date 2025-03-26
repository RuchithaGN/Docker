# Docker
Learn Docker concepts

In Docker, ENTRYPOINT defines the core executable that runs when a container starts, while CMD provides default arguments to that executable, which can be overridden by arguments passed during container execution. 
Here's a more detailed breakdown:

ENTRYPOINT:
•	Purpose: Specifies the command that the container will always execute when it starts. 
•	Behavior: The command specified by ENTRYPOINT is not replaceable by arguments passed to docker run. Instead, arguments passed to docker run are appended to the ENTRYPOINT command as arguments. 
•	Example:

    FROM ubuntu
    ENTRYPOINT ["/usr/bin/my_script"]
In this case, /usr/bin/my_script will always be executed when the container starts. 

CMD:
•	Purpose: Provides default arguments to the ENTRYPOINT command, or if no ENTRYPOINT is defined, the command to be executed.
•	Behavior: The arguments specified by CMD can be overridden by arguments passed to docker run.
•	Example: 

    FROM ubuntu
    ENTRYPOINT ["/usr/bin/my_script"]
    CMD ["--default-arg1", "--default-arg2"]
In this case, /usr/bin/my_script will be executed with --default-arg1 and --default-arg2 as arguments by default. However, if you run the container with docker run <image> --arg1 <new_arg1> --arg2 <new_arg2>, the ENTRYPOINT command will still be executed, but with --arg1 <new_arg1> and --arg2 <new_arg2> instead of the default arguments. 
Key Differences Summarized:
Feature	ENTRYPOINT	CMD
Purpose	Defines the main executable	Provides default arguments to ENTRYPOINT (or the main command if no ENTRYPOINT exists)
Override	Cannot be overridden by arguments passed to docker run	Can be overridden by arguments passed to docker run
Usage	Always executed when the container starts	Provides default arguments that can be changed when running the container

DISTROLESS IMAGES

Distroless images, designed for minimal size and enhanced security, are often used in conjunction with multi-stage builds to create smaller, more secure container images, by separating build and runtime environments. 
Here's a breakdown of how they work together:
Distroless Images:
•	Minimalist Approach:
Distroless images contain only the essential dependencies required to run a specific application, without including a full-fledged operating system distribution, package manager, or shell. 
•	Security Focus:
By removing unnecessary components, distroless images reduce the potential attack surface and vulnerabilities, making them ideal for production environments. 
•	Smaller Image Size:
The minimal nature of these images results in smaller image sizes, leading to faster image pull times, reduced storage costs, and quicker deployments. 

Multi-Stage Builds:

•	Separation of Concerns:
Multi-stage builds allow you to separate the build environment from the runtime environment in your Dockerfile. 
•	Build Stage:
The first stage (or stages) can be used for building and testing the application, including installing build tools and dependencies. 
•	Runtime Stage:
The final stage uses a distroless image as the base, and only copies the necessary application artifacts (like compiled binaries or libraries) from the build stage, resulting in a minimal runtime image. 
•	Benefits:
This approach leads to smaller final images, improved security, and efficient resource utilization. 
How They Work Together:
1.	Choose a Distroless Image:
Select a distroless image based on your application's requirements (e.g., distroless/python, distroless/go). 
2.	Multi-Stage Dockerfile:
Write a Dockerfile that includes multiple stages:
•	A build stage using a standard Linux distribution with build tools. 
•	A runtime stage using the distroless image as the base. 
3.	Copy Artifacts:
Use the COPY command to copy only the necessary application artifacts from the build stage to the runtime stage. 
4.	Build and Run:
Build the Docker image using docker build and run the container using docker run.

Example: 1
############################################
# Stage 1: Build stage (using a standard Linux distribution)
FROM ubuntu:latest AS builder

RUN apt-get update && apt-get install -y --no-install-recommends <build-tools>

WORKDIR /app

COPY . .

RUN <build-commands>

# Stage 2: Runtime stage (using a distroless image)
FROM distroless/python3-debian12 AS runtime

WORKDIR /app

COPY --from=builder /app/ <application-artifacts>

CMD ["python3", "<application-entrypoint>"]

##########################################
Example: 2
###########################################
# BASE IMAGE
###########################################

FROM ubuntu AS build

RUN apt-get update && apt-get install -y golang-go

ENV GO111MODULE=off

COPY . .

RUN CGO_ENABLED=0 go build -o /app .

############################################
# HERE STARTS THE MAGIC OF MULTI STAGE BUILD
############################################

FROM scratch

# Copy the compiled binary from the build stage
COPY --from=build /app /app

# Set the entrypoint for the container to run the binary
ENTRYPOINT ["/app"]
############################################
![image](https://github.com/user-attachments/assets/27760657-cdcd-463a-adc1-3588293ff6a1)

Benefits of using Distroless Images with Multi-Stage Builds:
•	Reduced Image Size:
Smaller images are easier and faster to deploy and update. 
•	Enhanced Security:
Fewer components in the image mean a smaller attack surface and fewer vulnerabilities. 
•	Improved Efficiency:
Smaller images consume fewer resources, leading to better resource utilization. 
•	Simplified Dependency Management:
Clearer dependency management, as only necessary dependencies are included. 

![image](https://github.com/user-attachments/assets/39f313c0-bef3-4549-b552-694d534c679e)
