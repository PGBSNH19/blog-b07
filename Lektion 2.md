## 2020-09-03

### SWAT analys av Virtuell Maskin mot Fysisk Maskin

### Styrkor

Med en VM kan man snabbt konfigurera och sätta upp sitt system för de syften man vill använda den, och kan göra så både genom GUI eller bash commands (SSH). Man behöver allokera en del av värd datorn, och kan göra så flera gånger, alltså köra flera VM på en värd. Dessa Virtuella Maskiner erbjuder en utmärkt miljö för att installera och provköra olika program och appar, även att testa saker i en säker miljö, och fungerar även som servers.

En fördel är att man kan sätta upp flera sepparata VM på en host som inte känner till varandra, alltså kan de arbeta fullständigt individuellt.

### Svagheter

Tyvärr är en VM långsammare än en fysisk maskin främst med tanke på optimering. En värddators OS känner sig själv bättre en än VM, som i många fall är en stor fil som behöver genomarbetas. 

Dom medför dessutom alltid overhead vilket innebär att de inte kan nå samma prestanda som en fysisk maskin. Så även om de har samma specs kommer en VM alltid att vara underlägsen. 

En lokal VM tar upp mycket plats (särskilt RAM) på värddatorn.

### Möjligheter

Möjligheterna med VM är många, de har inga fysiska begränsningar och är bara begränsade till hur mycket plats och prestanda en värddator har. Dom erbjuder möjligheten att testa saker i en separat och skyddad miljö, dessutom att testa program som är plattforms-specifika - till exempel kan man testa Linux och MAC program på en Windows värddator.  

Med hjälp av molnet kan man nå cloud hostade VMs oberoende av plats.

### Risker

En virtuell maskin som ägs av någon annan *(cloud hosted)* kan medföra risken att någon annan kan se och har tillgång till ens innehåll. En sådan löper också större risk för anfall *(data breach)*, man kan få sina kontouppgifter stulna med mera. En del av dessa risker kan minimeras, eller åtminstone ges bättre kontroll över ifall man kör en lokal Virtuell Maskin. 


### Virtuell Maskin med Azure CLI

Första steget var att vi skaffade learnet konton till Azure Portal och laddade ner Azure CLI. Via en windows terminal loggar man sedan in på Azure med `az login` vilket tar en till en prompt för inloggning via webblässare. Sedan skapa en resursgrupp: 
`az group create --name testResourceGroup --location northeurope`. Generara en VM görs sedan med:
`az vm create --resource-group testResourceGroup  --name testVM --image UbuntuLTS  --admin-username azureuser --generate-ssh-keys`

Med vad man har gjort har nu 7 resurser skapats i Azure Portal, och för att logga in på vår VM kan vi skriva:
`ssh azureuser@GivenIPAdress`


### Pulumilösning 

Vi började att experimentera oss fram och laddade ner exmepelrepot ifrån `https://github.com/pulumi/examples`
I terminalen går vi in i mappen ./azure-cs-webserver och loggar in på azure: 
`az login`
Vi sätter en plats(*location*) för Azure: `pulumi config set azure:location westus`
och använder oss av *westus* för enkelhets skull, annars kan vi också ange **northeu** och får då ändra i WebServerStack.cs, egenskapen VmSize från `Standard_A2` till `Standard_A2_v2`.
`pulumi up` grupperar vår VM på Azures server, och vi ser en VM dyka upp. 
Namn på datorn, resourcegroup med mera är delvis angivet i WebServerStack.cs men genereras också med slumpade nycklar, så för att hitta de exakta namnen går vi in på Azure Portal.

Följande steg, för att få SimpleWebHalloWorld skriptet att rulla på servern är väldigt snarlika de från tidigare uppgifterna. Vi öppnar först port 80 för webbtraifk: 
`az vm open-port --port 80 --resource-group server-rg<random key> --name server-vm<random key>`
Sedan loggar in med SSH: `ssh testadmin@<given ip address>` public IP address hittar vi i Azure Portal

Man promptas för lösenord och skriver in lösenordssträngen, och vips så är vi inloggade på vår Pulumi genererade VM. 

För att få SimpleWebHalloWorld att rulla så laddar vi först hem scriptet: 
`git clone https://github.com/skjohansen/...git`
Sedan behöver vi .NET SDK och .NET Core Runtime, bägge hämtas via terminalen: 
`sudo apt-get ...` 
Nu har vi allt vi behöver, vi cdar in i det lokala repot: 
`cd ./SimpleWebHalloWorld/`
och för att kompilera kör kommandot: 
`dotnet publish --configuration Release`
vi cdar in till publish folder: 
`cd .//bin/Release/netcoreapp3.1/publish/` 
och kan sedan köra 
`dotnet SimpleWebHalloWorld.dll --urls http://0.0.0.0:5000`

Webbapplikationen är då publiserad och körs på vår VM och ska kunna hittas på den publika IP addressen.
