2020-09-09

### vad en är pipeline?

En pipeline är en uppsättning av automatiserade processer som låter utvecklaren att kompilera, builda och distribuera deras kod till produktion. 

en (pipeline.yml) är en fil med instruktioner om hur datorn ska utföra automatiserade operationer.

vår yml fil innehåller denna instrktionen for att köra automatiserade tester. Vi har skapat trigger för master branchen, det betyder att så fort ny commit kommer till vår github repository på master branchen utförs test operationen eftersom vi har bestämt det med (command:test)

#####################################################

trigger:
- master

pool:
  vmImage: 'ubuntu-latest'


variables:
  buildConfiguration: 'Release'

steps:
- task: DotNetCoreCLI@2
  inputs: 
    command: test
    project: '/Pipeline.test/*.csproj'
    arguments: '--configuration $(buildConfiguration)'
- script: dotnet build --configuration $(buildConfiguration)
  displayName: 'dotnet build $(buildConfiguration)'

#####################################################

###Instruktioner för att skapa automatiserade tester:

1- öppna Git-bash 

-----------------------

2- skapa en mapp och navigera i det

mkdir myMapp

cd myMapp/

--------

3- skapa ny mvc projekt

dotnet new MVC

---------------------

4- öppna projektet med visual studio

--------------------

5- höger klick på projektfilen och välj add > Docker-support

detta kommer skapa en Dockerfil för din projekt

---------------------

6- gå tillbaka till Git-bash och skapa azure contianer registry du måste ha Azure CLI för det. om du inte har ladda ner det från nätet

---------------

7- az login

du kommer bli navigerat till azure websida för att logga in

--------------

8- Nu kan du skapa en Resource group på azure med command line från gitbash

az group create --name samirResource --location northeurope

-------------

9- skapa en container registry

 az acr create --resource-group samirResource --name samirreg --sku basic

----------------

10- googla azure DevOps och login där sedan skapa en organisation

---------------------

11- skapa en ny projekt

---------------

12-gå till project settings > service connections

-------

13- skapa ny service connection

välj Docker registry > azure container registry

fill i raderna med följande

Subscription:      	 din subscription id från azure portal

azure container registry: 		namn på din container registry

service connection name:     		välj en service connection namn

------

14- skapa en pipeline

klick på pipepile > new pipeline > gitHub > välj din projekt repository på gitHub > starter pipeline 

------

15- klick på din pipeline  och klick på edit

-----------

16- för att automatisk köra test för varje push till master branchen på github skriv denna kod. 
kom ihåg att det är viktigt indentering i yml filer

##############################################

trigger:

- master

pool:

 vmImage: 'ubuntu-latest'

variables:

 buildConfiguration: 'Release'

steps:

- task: DotNetCoreCLI@2

 inputs: 

  command: test

  project: '/Pipeline.test/*.csproj'      ( namnen för test projekt)

  arguments: '--configuration $(buildConfiguration)'

- script: dotnet build --configuration $(buildConfiguration)

 displayName: 'dotnet build $(buildConfiguration)'

###############################################

17- för att pusha koden till azure container registry

skapa ny pipeline och lägg in denna koden

######################################################

trigger:

- master

resources:

- repo: self

variables:

 imageRepository: 'samirreg.azurecr.io'    (namn på din container resgistry.azurecr.io)

 containerRegistry: 'samirreg'    (namn för din container registry)

 tag: '$(Build.BuildId)'

 vmImageName: 'ubuntu-latest'

stages:

- stage: Build

 displayName: Build and push stage

 jobs: 

 - job: Build

  displayName: Build

  pool:

   vmImage: $(vmImageName)

  steps:

  - task: Docker@2

   displayName: Build and push an image to container registry

   inputs:

​    command: buildAndPush

​    repository: $(imageRepository)

​    dockerfile: Dockerfile      (namn för din dockerfil*ange  relativ sökvägen *)

​    containerRegistry: samirreg-acr      (service connection name som vi la p project settings)

​    tags: |

​     $(tag)
