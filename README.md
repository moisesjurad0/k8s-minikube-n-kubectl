# k8s-minikube-n-kubectl

This workshop will include steps for using minikube & kubectl on Windows 11

links reference:

- <https://minikube.sigs.k8s.io/docs/start/>
- <https://minikube.sigs.k8s.io/docs/drivers/>
- <https://minikube.sigs.k8s.io/docs/drivers/docker/>

previous steps:

1. install docker for Windows

steps:

1. run in terminal: `winget install minikube`
1. run in terminal: `minikube start --driver=docker`
    - error output:

    ```txt
    minikube: The term 'minikube' is not recognized as a name of a cmdlet, function, script file, or executable program.
    Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
    ```

    - if you have this error try to restart the windows terminal or follow these link(s):
        - <https://minikube.sigs.k8s.io/docs/drivers/>
            - <https://minikube.sigs.k8s.io/docs/drivers/docker/>
                - <https://minikube.sigs.k8s.io/docs/tutorials/wsl_docker_driver/>
    - success output:

        ![Image01](<https://github.com/moisesjurad0/k8s-minikube-n-kubectl/raw/main/images/Screenshot 2023-07-19 211348.png>)

        ![Image02](<https://github.com/moisesjurad0/k8s-minikube-n-kubectl/raw/main/images/Screenshot 2023-07-19 205259.png>)

1.
