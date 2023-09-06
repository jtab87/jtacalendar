# Plugins budibase

## Installer (docker) de budibase 

cf Guillaume

Dans le répertoire **C:\produits\budibase**, on place les fichiers **.env** et **docker-compose.yaml** récupérés dans l'install docker

## Installer (si besoin) la dernière version de Budibase CLI 

```
npm install -g @budibase/cli
```

## Installer (si besoin) yarn 

```
npm install -g yarn
```

## On crée un répertoire pour les plugins 

C:\produits\budibase\plugins

## On modifie les fichiers de configuration docker

### <u>docker-compose.yaml</u>

On ajoute le répertoire des plugins (et son point de montage) au app-service volumes
```
app-service:	
    volumes:
      - ./plugins:/plugins
```	  

### <u>.env</u>

On renseigne la variable **PLUGINS_DIR** avec le point de montage spécifié dans **docker-compose.yaml**
```
PLUGINS_DIR=/plugins
```

## Initialisation du plugin

On initialise le plugin dans le répertoire des plugins
```
cd C:\produits\budibase\plugins
budi plugins --init component
```
et on renseigne les infos demandées (name, description, version, ...)

un répertoire est créé avec le nom du plugin

After you have created your new plugin directory, execute the following:
```
cd <le_répertoire_du_composant>
yarn build
```

## démarrer budibase

```
cd C:\produits\budibase
docker-compose up -d    (-d pour "mode demon", sinon y rend pas la main, et il faudra faire CTRL C pour stopper budibase)
```
Le nouveau plugin doit maintenant apparaitre dans la liste des plugins installés

## Arréter budibase (si besoin)

```
cd C:\produits\budibase
docker-compose stop
```

# Jtacalendar
This is a readme for your new Budibase plugin.

# Description
Super calendar jta

Find out more about [Budibase](https://github.com/Budibase/budibase).

## Instructions

To build your new  plugin run the following in your Budibase CLI:
```
budi plugins --build
```

You can also re-build everytime you make a change to your plugin with the command:
```
budi plugins --watch
```
