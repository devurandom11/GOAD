# GOAD

- LAB Content 

![GOAD overview](../../docs/img/GOAD_schema.png)

## Servers
This lab is actually composed of five virtual machines:
- **capsulecorp** : DC01  running on Windows Server 2019 (with windefender enabled by default)
- **namekplanet**   : DC02  running on Windows Server 2019 (with windefender enabled by default)
- **lookout**  : SRV02 running on Windows Server 2019 (with windefender **disabled** by default)
- **beerusplanet**      : DC03  running on Windows Server 2016 (with windefender enabled by default)
- **kameisland**      : SRV03 running on Windows Server 2016 (with windefender enabled by default)

## domain : namek.earthrealm.local
- **namekplanet**     : DC01
- **lookout**    : SRV02 : MSSQL / IIS

## domain : earthrealm.local
- **capsulecorp**   : DC02
- **gizardwastelands**     : SRV01 (disabled due to resources reasons)

## domain : universe7.local
- **kameisland**        : DC03
- **beerusplanet**         : SRV03 : MSSQL / ADCS

The lab setup is automated using vagrant and ansible automation tools.
You can change the vm version in the Vagrantfile according to Stefan Scherer vagrant repository : https://app.vagrantup.com/StefanScherer


## Users/Groups and associated vulnerabilites/scenarios

- You can find a lot of the available scenarios on [https://mayfly277.github.io/categories/ad/](https://mayfly277.github.io/categories/ad/)

- Graph of some scenarios is available here :
![diagram-GOAD_compromission_Path_dark](./../../docs/img/diagram-GOAD_compromission_Path_dark.png)

NAMEK.EARTHREALM.LOCAL
- NAMEKIANS:              RDP on NAMEKPLANET AND LOOKOUT
  - gohan:        Execute as user on mssql, pass on all share
  - piccolo:      DOMAIN ADMIN NAMEK/ (bot 5min) LLMRN request to do NTLM relay with responder
  - nail:     
  - dende:        bot (3min) RESPONDER LLMR / lsass present user
  - videl:       keywalking password / unconstrained delegation
  - chiaotzu:     ASREP_ROASTING
  - tien:      pass spray WinterYYYY
  - uub:          mssql admin / KERBEROASTING / mssql trusted link
  - popo:             PASSWORD SPRAY (user=password)
- LOOKOUTS:         RDP on LOOKOUT
  - yajirobe:     Password in ldap description / mssql execute as login
                       GPO abuse (Edit Settings on "NAMEKWALLPAPER" GPO)
  - uub:          (see namekians)
  - kingkai:      (see kingkaistudents)
- KINGKAI:             RDP on LOOKOUT
  - kingkai:      Admin lookout, pass in sysvol script
- RealmTravel :       cross forest group

EARTHREALM.LOCAL
- ZFIGHTERS
  - vegeta:   ACE forcechangepassword on yamcha, password on sysvol cyphered
  - yamcha:   ACE genericwrite-on-user trunks
  - krillin:   ACE self membership on earthdefenders
  - bulma:  DOMAIN ADMIN EARTHREALM
- SAIYANS:           RDP on CAPSULECORP
  - goku:  DOMAIN ADMIN EARTHREALM, protected user
  - trunks: ACE Write DACL on krillin
  - goten:   WriteDACL on container, sensitive user
  - gotenks: ACE genericall-on-computer capsulecorp 
- EARTHDEFENDERS :      ACE add Member to group elitewarriors / RDP on CAPSULECORP
  - roshi:    
  - jaco:        ACE genericall-on-group Domain Admins and sdholder
  - korin:   
- ELITEWARRIORS :        ACE Write Owner on group DEITYGUARDS
- DEITYGUARDS :         ACE generic all on user gotenks
- UniverseTravel:       cross forest group

UNIVERSE7.LOCAL
- GODOFDESTRUCTION
  - mai :          ASREP roasting, generic all on frieza
  - beerus: DOMAIN ADMIN UNIVERSE7
  - whis:  ACE write property on zarbon
  - zarbon:      mssql execute as login / mssql trusted link / Read LAPS Password
- FRIEZAFORCE
  - frieza:         mssql admin / GenericAll on whis (shadow credentials) / GenericAll on ECS4
- Z-ALLIES:       cross forest group
- Observers:                 cross forest group / Read LAPS password  / ACL generic all zarbon

## Computers Users and group permissions

- EARTHREALM
  - DC01 : capsulecorp.earthrealm.local (Windows Server 2019) (EARTHREALM DC)
    - Admins : goku (U), bulma (U)
    - RDP: EarthDefenders (G)

- NAMEK
  - DC02 : namekplanet.namek.earthrealm.local (Windows Server 2019) (NAMEK DC)
    - Admins : piccolo (U), nail (U), dende (U)
    - RDP: Namekians(G)

  - SRV02 : lookout.universe7.local (Windows Server 2019) (IIS, MSSQL, SMB share)
    - Admins: kingkai (U)
    - RDP: Lookouts (G), KingKai (G), Namekians (G)
    - IIS : allow asp upload, run as NT Authority/network
    - MSSQL:
      - admin : uub
      - impersonate : 
        - execute as login : yajirobe -> sa
        - execute as user : gohan -> dbo
      - link :
        - to kameisland : uub -> sa

- UNIVERSE7
  - DC03  : beerusplanet.universe7.local (Windows Server 2016) (UNIVERSE7 DC)
    - Admins: beerus (U)
    - RDP: GodOfDestruction (G)

  - SRV03 : kameisland.universe7.local (Windows Server 2016) (MSSQL, SMB share)
    - Admins: frieza (U)
    - RDP: FriezaForce (G)
    - MSSQL :
      - admin : frieza
      - impersonate :
        - execute as login : zarbon -> sa
      - link:
        - to lookout: zarbon -> sa

## Blueteam / ELK

- **elk** a kibana is configured on http://192.168.56.50:5601 to follow the lab events
- infos : log encyclopedia : https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/
- the elk is not installed by default due to resources reasons. 

- prerequistes: 
- you need `sshpass` for the elk installation
```bash
sudo apt install sshpass
```
- Chocolatey is needed to use elk. To install it run:
```bash
ansible-galaxy collection install chocolatey.chocolatey 
```

- To install and start the elk play the following commands :
```bash
./goad.sh -t install -l GOAD -p virtualbox -m local -r elk.yml -e elk
#or
./goad.sh -t install -l GOAD -p vmware -m local -r elk.yml -e elk
```

 * -e : to add the elk in vagrantfile for the vagrant up
 * -r : to run only the elk.yml ansible script

If you want to only launch the elk.yml file without providing (vm creation), because your elk vm is already up do :
```bash
./goad.sh -t install -l GOAD -p virtualbox -m local -a -r elk.yml -e elk
#or
./goad.sh -t install -l GOAD -p vmware -m local -a -r elk.yml -e elk
```

If you want to do that by hand:
  1. run: `GOAD_VAGRANT_OPTIONS=elk vagrant up`

  2. launch the elk provisionning with the command :
  ```
  cd ansible
  ansible-playbook -i ../ad/GOAD/data/inventory -i ../ad/GOAD/providers/<your_provider>/inventory elk.yml
  ```

### V2 breaking changes
- If you previously install the v1 do not try to update as a lot of things have changed. Just drop your old lab and build the new one (you will not regret it)
- Chocolatey is no more used and basic tools like git or notepad++ are no more installed by default (as chocolatey regularly crash the install due to hitting rate on multiples builds)
- ELK is no more installed by default to save resources but you still can install it separately (see the blueteam/elk part)
- GizardWastelands vm as disappear and there is no more DC replication in the lab to save resources
- Namekplanet is now a domain controller for the subdomain namek of the earthrealm.local domain

