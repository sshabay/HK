Azure-testmiljö för Sweditech AB

Ett skolprojekt i kursen Cloud Services (Cloud- och Infrastrukturspecialist, YH, EC Utbildning). Jag byggde en liten testmiljö i Microsoft Azure åt det fiktiva bolaget Sweditech AB (ca 50 anställda) som överväger att flytta delar av sin lokala IT till molnet.

Miljön är en lärandemiljö. Inga lösenord, nycklar eller anslutningssträngar finns i repot, och alla skärmbilder är maskerade.

Mål

Bygga en minimal men fungerande miljö som visar grunderna i IaaS-drift i Azure:

resursorganisation och namngivning
virtuellt nätverk och nätverkssäkerhet
två virtuella maskiner som kommunicerar
delad lagring
en databasserver som nås från en annan VM
Arkitektur
Resursgrupp: sweditech-rg-test (Sweden Central)
│
├── VNet: sweditech-vnet-test (10.0.0.0/16)
│     └── Subnät: sweditech-subnet-vm (10.0.0.0/24)
│           ├── VM: sweditech-vm-sql  (Windows Server, B2ats_v2)  ← SQL Server
│           └── VM: sweditech-vm-app  (Windows Server, B2ats_v2)  ← klient/applikation
│
├── NSG per VM (RDP 3389 + Azures standardregler)
└── Storage Account: sweditechstoragetest (LRS)
      └── Filresurs: sweditech-share (100 GB) → monterad som Z: på båda VM:arna
Vad som byggdes
Område	Lösning
Resursgrupp	sweditech-rg-test, region Sweden Central, samlar alla resurser så att de kan hanteras och tas bort tillsammans
Nätverk	VNet 10.0.0.0/16 med ett subnät 10.0.0.0/24 för VM:arna
Nätverkssäkerhet	Nätverkssäkerhetsgrupper (NSG) med regel för RDP (TCP 3389) samt Azures standardregler
Virtuella maskiner	Två Windows Server-VM:ar (B2ats_v2) i samma subnät
Lagring	Storage Account (LRS) med filresurs på 100 GB, monterad som enhet Z: på båda VM:arna
Databas	SQL Server 2019 (Evaluation) på sweditech-vm-sql, nåbar via SSMS från sweditech-vm-app
Verifiering och felsökning

Varje steg testades, och några misslyckade försök blev bra lärdomar:

Ping mellan VM:arna. Första testet gav Request timed out. Orsaken var att Windows-brandväggen blockerar inkommande ICMP som standard. Jag löste det med en brandväggsregel via PowerShell, och sedan blev det 0 % paketförlust.
Delad lagring. Jag skapade test.txt på den ena VM:en och bekräftade att filen syntes på den andra.
SQL Server-anslutning.
Lokalt i SSMS kom först ett certifikatfel, som löstes i anslutningsinställningarna.
TCP/IP aktiverades på port 1433, med en brandväggsregel för porten.
Autentiseringsläget ändrades till SQL Server and Windows Authentication (Mixed Mode), och tjänsten startades om.
Ett inloggningskonto skapades. Fjärranslutningen från sweditech-vm-app gav först Error 18456 (inloggning misslyckades) och fungerade därefter mot 10.0.0.4.
Verktyg och teknik

Microsoft Azure (Portal, resursgrupper, VNet, NSG, VM, Storage Account/Azure Files) · Windows Server · PowerShell · SQL Server · SSMS

Det jag lärde mig
Hur Azures resurser hänger ihop: resursgrupp, nätverk, VM och lagring.
Att felsöka nätverk lager för lager: Azure NSG först, sedan Windows-brandväggen i gästen.
Skillnaden mellan IaaS, PaaS och SaaS, och shared responsibility-modellen i praktiken.
Att aldrig lägga hemligheter i dokumentation. Jag maskerade lagringsnycklar och annat känsligt.
Vad jag skulle göra annorlunda i en riktig miljö

Testmiljön är medvetet enkel. I produktion skulle jag förbättra:

RDP: begränsa till kända IP-adresser eller använda Azure Bastion/JIT-åtkomst i stället för öppen RDP.
SQL: använda Windows-autentisering eller Entra ID och minsta möjliga behörighet i stället för ett konto med sysadmin-roll.
Infrastructure as Code: beskriva hela miljön i Bicep eller Terraform så att den kan återskapas automatiskt.
Övervakning: aktivera Azure Monitor och loggning.
Kostnadskontroll: budgetar och aviseringar, samt att stoppa eller ta bort resurser när testet är klart.
Struktur i repot
.
├── README.md
├── docs/
│   └── images/        # maskerade skärmbilder (bild 1–21)
└── scripts/           # PowerShell (t.ex. brandväggsregeln för ICMP)
Om mig

HK, studerar Cloud- och Infrastrukturspecialist (YH). Intresserad av Windows Server, Azure och drift. Kontakt: via LinkedIn eller GitHub.
