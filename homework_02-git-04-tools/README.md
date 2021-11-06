##Домашнее задание к занятию «2.4. Инструменты Git» - Денис Поляков
####1. Найдите полный хеш и комментарий коммита, хеш которого начинается на `aefea`.
```
%git show -s aefea  
commit aefead2207ef7e2aa5dc81a34aedf0cad4c32545  
Author: Alisdair McDiarmid <alisdair@users.noreply.github.com>  
Date:   Thu Jun 18 10:29:58 2020 -0400

   Update CHANGELOG.md
```
####2. Какому тегу соответствует коммит 85024d3? 
```
% git show -s --oneline 85024d3  
85024d310 (tag: v0.12.23) v0.12.23
```
####3. Сколько родителей у коммита b8d720? Напишите их хеши.
```
%git log --pretty=format:'%h %s' --graph b8d720  
*   b8d720f83 Merge pull request #23916 from hashicorp/cgriggs01-stable  
|\  
| * 9ea88f22f add/update community provider listings  
|/  
*   56cd7859e Merge pull request #23857 from hashicorp/cgriggs01-stable  
```
У коммита  b8d720, полученного в рузультате мержа,  два родителя: **9ea88f22f и 56cd7859e**

Так же можно восопользоваться командой:
```
% git show b8d720^  
commit 56cd7859e05c36c06b56d013b55a252d0bb7e158  
Merge: 58dcac4b7 ffbcf5581  
Author: Chris Griggs <cgriggs@hashicorp.com>  
Date:   Mon Jan 13 13:19:09 2020 -0800  

    Merge pull request #23857 from hashicorp/cgriggs01-stable

   [cherry-pick]add checkpoint links
```

####4. Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами v0.12.23 и v0.12.24.
```
% git log --oneline v0.12.23..v0.12.24  
33ff1c03b (tag: v0.12.24) v0.12.24  
b14b74c49 [Website] vmc provider links  
3f235065b Update CHANGELOG.md  
6ae64e247 registry: Fix panic when server is unreachable  
5c619ca1b website: Remove links to the getting started guide's old location  
06275647e Update CHANGELOG.md  
d5f9411f5 command: Fix bug when using terraform login on Windows  
4b6d06cc5 Update CHANGELOG.md  
dd01a3507 Update CHANGELOG.md  
225466bc3 Cleanup after v0.12.23 release  
```
####5. Найдите коммит в котором была создана функция func providerSource, ее определение в коде выглядит так func providerSource(...) (вместо троеточего перечислены аргументы).
```
%git grep --break --heading -n 'func providerSource('  
provider_source.go  
23:func providerSource(configs []*cliconfig.ProviderInstallation, services *disco.Disco) (getproviders.Source, tfdiags.Diagnostics) {
```
Сомтрим в каком комите встречаются совпадения:
```
%git log -S'func providerSource(' --oneline  
8c928e835 main: Consult local directories as potential mirrors of providers
```
Ответ: **8c928e835** причем, если бы запрос был бы без `(`, а именно `func providerSource` в этом репо было бы уже два совпадения и два коммита при поиске. В этом случае пршлось бы или детализировать строку поиска, или при помощи `git show хэш` перебирать и визуально оценивать изменения в файлах.

####6. Найдите все коммиты в которых была изменена функция globalPluginDirs.  
Для поиска всех коммитов с этой функцией ищу в каком файле определялась функция: 
```
%git grep --break --heading -np 'globalPluginDirs'  
commands.go  
51=func initCommands(  
88:             GlobalPluginDirs: globalPluginDirs(),  
429=func credentialsSource(config *cliconfig.Config) (auth.CredentialsSource, error) {  
430:    helperPlugins := pluginDiscovery.FindPlugins("credentials", globalPluginDirs())    
    
internal/command/cliconfig/config_unix.go  
31=func homeDir() (string, error) {  
34:             // FIXME: homeDir gets called from globalPluginDirs during init, before    
  
plugins.go  
3=import (  
12:// globalPluginDirs returns directories that should be searched for  
18:func globalPluginDirs() []string {  
```
вижу, что поиск нужно делать в файле `plugins.go` определение было в нем в 18 строке.   
Делаю поиск в файле для оценки и просмотра изменений:
```
%git log -L :globalPluginDirs:plugins.go  
```
Для ответа на поставленный вопрос делаю вывод толко комимтов одной строкой:  
```
%git log -L :globalPluginDirs:plugins.go  -s --oneline    
78b122055 Remove config.go and update things using its aliases    
52dbf9483 keep .terraform.d/plugins for discovery    
41ab0aef7 Add missing OS_ARCH dir to global plugin paths    
66ebff90c move some more plugin search path logic to command    
8364383c3 Push plugin discovery down into command package    
```
####7.  Кто автор функции `synchronizedWriters`?
Смотрю в каких коммитах встречается функция:
```
%git log -S'synchronizedWriters'  --oneline  
bdfea50cc remove unused  
fd4f7eb0b remove prefixed io  
5ac311e2a main: synchronize writes to VT100-faker on Windows  
```
Смотрю первый коммит с ее упоминанием (в последнем она была удалена):
```
%git show 5ac311e2a  
```
Вижу добавление ее в файл synchronized_writers.go: 
```
+func synchronizedWriters(targets ...io.Writer) []io.Writer {`  
```
Автор этого 5ac311e2a коммита  **Martin Atkins <mart@degeneration.co.uk>**

