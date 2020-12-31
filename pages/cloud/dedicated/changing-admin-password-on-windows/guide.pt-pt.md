---
title: 'Alterar a palavra-passe administrador num servidor dedicado Windows'
slug: "alterar-palavra-passe-admin-windows"
excerpt: 'Saiba como alterar a palavra-passe de um administrador num servidor dedicado Windows'
section: 'Diagnóstico e Modo Rescue'
---

**Última atualização: 16 de dezembro de 2020**

> [!primary]
> Esta tradução foi automaticamente gerada pelo nosso parceiro SYSTRAN. Em certos casos, poderão ocorrer formulações imprecisas, como por exemplo nomes de botões ou detalhes técnicos. Recomendamos que consulte a versão inglesa ou francesa do manual, caso tenha alguma dúvida. Se nos quiser ajudar a melhorar esta tradução, clique em "Contribuir" nesta página.
>

## Objetivo

Quando instalar ou reiniciar um sistema operativo Windows, irá receber uma palavra-passe de acesso administrador. Para maior segurança, sugerimos a alteração da mesma, conforme as indicações apresentadas no manual "[Proteger um servidor dedicado](../proteger-um-servidor-dedicado/){.external}". Se perdeu a palavra-passe admin, deverá reinicializá-la em modo rescue.

**Este manual explica-lhe como alterar a palavra-passe "admin" do seu servidor através das configurações de modo rescue disponíveis para um sistema operativo Windows.**

## Requisitos

* Possuir um [servidor dedicado](https://www.ovhcloud.com/pt/bare-metal/){.external} com o Windows instalado.
* Ter acesso à [Área de Cliente OVHcloud](https://www.ovh.com/auth/?action=gotomanager){.external}.


## Instruções

As etapas seguintes descrevem o processo de modificação da palavra-passe admin local através do modo rescue OVHcloud (baseado em Linux) que está disponível a qualquer momento. Se preferir utilizar o Windows PE (WinRescue), consulte o método dedicado [no fim deste guia](./#reinicializacao-da-password-admin-com-o-auxilio-de-winrescue_1). 

### 1 - reiniciar o servidor em modo rescue

O sistema deve ser ativado em modo rescue antes de poder alterar a palavra-passe admin. Aceda à Área de [Cliente OVHcloud](https://www.ovh.com/auth/?action=gotomanager), aceda à secção `Bare Metal Cloud`{.action} e selecione o seu servidor na lista de navegação à esquerda em `Servidores dedicados`{.action}.

O netboot deve ser migrado para "rescue64-pro (Customer rescue system (Linux)". Procure "Boot" na zona **Informações gerais** e clique em `...`{.action} e depois em `Alterar`{.action}.
<br>Na janela que aparecer, selecione **Fazer boot em modo rescue** e selecione "rescue64-pro" no menu. Indique um endereço de e-mail no último campo se os dados de acesso tiverem de ser enviados para um endereço diferente do endereço principal da sua conta OVHcloud. 

Clique em `Seguinte`{.action} e depois em `Validar`{.action}.

![prazo](images/adminpw_win_01.png){.thumbnail}

Concluída a alteração, clique em `...`{.action} à direita de "Status" na zona **Estado dos serviços**.
<br>Clique em `Reiniciar`{.action} e o servidor será reiniciado em modo rescue. Esta operação pode demorar alguns minutos.
<br>Pode verificar o progresso sob o separador `Tarefas`{.action}. Receberá um e-mail com os ID (incluindo a palavra-passe de ligação) do utilizador "root" do modo rescue.

![rescuereboot](images/adminpw_win_02.png){.thumbnail}

Para mais informações sobre o modo rescue, consulte [este guia](../rescue_mode/).

### 2 - Montar a partição do sistema

Ligue-se ao seu servidor através de SSH. Caso seja necessário, consulte o guia da OVHcloud.
<br> Como se trata de um servidor Windows, as partições serão intituladas "Microsoft LDM data".

```
# fdisk -l
Disk /dev/sda: 1.8 TiB, 200398934016 bytes, 3907029168
Units: sectores de 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel tipo: gpt
Disk identifier: 54A5B25A-75B9-4355-9185-8CD958DCF32A
 
Device Start End Sectors Size Type
/dev/sda1        2048     718847     716800  350M EFI System
/dev/sda2 718848 720895 2048 1M Microsoft LDM metadata
/dev/sda3 720896 980991 260096 127M Microsoft reserved
/dev/sda4 980992 3907028991 3906048000 1.8T Microsoft LDM data
/dev/sda5 3907028992 3907029134 143 71.5K Microsoft LDM data
```

Neste exemplo, "sda4" é a partição do sistema, determinada pelo seu tamanho. Geralmente, existe também uma segunda partição espelho que, neste caso, se chama "/dev/**sdbX**". Na maioria dos casos, o servidor terá vários discos com esquemas de partição idênticos. Para o processo de reinicialização da palavra-passe, apenas o primeiro é importante. 

Agora, monte esta partição:

```
# mount /dev/sda4 /mnt
```

Verifique o ponto de montagem:

```
# lsblk
NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sdb 8:16 0 1.8T 0 disk
├ ─ sdb4 8:20 0 1.8T 0 parte
├ ─ sdb2 8:18 0 1M 0 parte
├ ─ sdb5 8:21 0 71.5K 0 parte
├ ─ sdb3 8:19 0 127M 0 parte
└ ─ sdb1 8:17 0 350M 0 parte
sda 8:0 0 1.8T 0 disk
├ ─ sda4 8:4 0 1.8T 0 parte /montante
├ ─ sda2 8:2 0 1M 0 parte
├ ─ sda5 8:5 0 71.5K 0 parte
├ ─ sda3 8:3 0 127M 0 parte
└ ─ sda1 8:1 0 350M 0 parte
```

No exemplo acima, a operação foi bem-sucedida. Se a montagem falhou, receberá provavelmente uma mensagem de erro semelhante a esta: 

```
The disk contains an unclean file system (0, 0).
Metadata kept em Windows cache, recusa-te a mount.
Failed to mount '/dev/sda4': Operation not permitted
The NTFS partition is in an safe state. Plase resume e shutdown
Windows fully (no hibernação ou fast restarting), ou mount the volume
read-only with the 'ro' mount opção.
```

Neste caso, utilize o seguinte comando e tente novamente montar a partição.

```
# ntfsfix /dev/sda4
# mount /dev/sda4 /mnt
```

### Etapa 3: eliminar a password atual

Esta etapa consiste em manipular o ficheiro *SAM* com a ajuda de uma ferramenta que permite eliminar a palavra-passe do utilizador admin. Aceda à pasta correta e liste os utilizadores Windows:

```
# cd /mnt/Windows/System32/config
/mnt/Windows/System32/config# chntpw -l SAM

Chntpw version 1.00 140201, (c) Petter N Hagen
Hive <SAM> name (from header): <\SystemRoot\System32\Config\SAM>
ROOT KEY at offset: 0x001020 * Subkey indexing type is: 686c <lh>
File size 65536 [10000] bytes, containing 8 páginas (+ 1 headerpage)
Used for data: 359/39024 blocks/bytes, unused: 33/18064 blocks/bytes.

| RID -| Username - Admin? | Lock? —|
| 03e8 | admin                          | ADMIN  | dis/lock |
| 01f4 | Administrator                  | ADMIN  | dis/lock |
| 01f7 | DefaultAccount                 |        | dis/lock |
| 01f5 | Guest                          |        | dis/lock |
| 01f8 | WDAGUtilityAccount             |        | dis/lock |
```

Apagar a palavra-passe do utilizador admin através do seguinte comando. (Escolha "Administrator" se "admin" não existir.)

```
# chntpw -u admin SAM
Chntpw version 1.00 140201, (c) Petter N Hagen
Hive <SAM> name (from header): <\SystemRoot\System32\Config\SAM>
ROOT KEY at offset: 0x001020 * Subkey indexing type is: 686c <lh>
File size 65536 [10000] bytes, containing 8 páginas (+ 1 headerpage)
Used for data: 361/39344 blocks/bytes, unused: 35/13648 blocks/bytes.
 
==========================================================================================================================================================================================================================================================
 
RID: 1000 [03e8]a
Nome de utilizador: admin
fullname:
como:
homedir:
 
00000221 = Users (which has 3 membros)
00000220 = Administradores (which has 2 membros)
 
Account bits: 0x0010 =
[ ] Disabled        | [ ] Homedir req.    | [ ] Passwd not req. |
[ ] Temp. duplicate | [X] Normal account  | [ ] NMS account     |
[ ] Domain trust ac | [ ] Wks trust act.  | [ ] Srv trust act   |
[ ] Pwd don't exir | [ ] Auto lockout    | [ ] (unknown 0x08)  |
[ ] (unknown 0x10)  | [ ] (unknown 0x20)  | [ ] (unknown 0x40)  |
 
Failed login count: 0, while max tries is: 0
Total login em curso: 5
 
- - - User Edit Menu:
 1 - Clear (blank) user password
(2 - Unlock and enable user account) [seems unlocked already]
 3 - Promote user (make user an administrator)
 4 - Add user to a group
 5 - Remove user from a group
 q - Quit editing user, back to user select
Seleção: [q] >
```

Carregue "1" e carregue em Enter ( ↩). (Utilize a opção 2 se aparecer um X na frente de "Disabled".)

```
Seleção: [q] > 1
Password cleared!
==========================================================================================================================================================================================================================================================
 
RID: 1000 [03e8]
Nome de utilizador: admin
fullname:
como:
homedir:
 
00000221 = Users (which has 3 membros)
00000220 = Administradores (which has 2 membros)
 
Account bits: 0x0010 =
[ ] Disabled        | [ ] Homedir req.    | [ ] Passwd not req. |
[ ] Temp. duplicate | [X] Normal account  | [ ] NMS account     |
[ ] Domain trust ac | [ ] Wks trust act.  | [ ] Srv trust act   |
[ ] Pwd don't exir | [ ] Auto lockout    | [ ] (unknown 0x08)  |
[ ] (unknown 0x10)  | [ ] (unknown 0x20)  | [ ] (unknown 0x40)  |
 
Failed login count: 0, while max tries is: 0
Total login em curso: 5
** No NT MD4 hash found. This user Probably has a BLANK password!
** No LANMAN hash found either. Try login com o password!
 
- - - User Edit Menu:
 1 - Clear (blank) user password
(2 - Unlock and enable user account) [seems unlocked already]
 3 - Promote user (make user an administrator)
 4 - Add user to a group
 5 - Remove user from a group
 q - Quit editing user, back to user select
Seleção: [q] >
```

Carregue "q" e carregue em Enter para sair da ferramenta. Introduza "y" quando for convidado a fazê-lo e carregue em Enter.

```
Seleção: [q] > q
 
Hives that have changed:
 # Name
 0  <SAM>
Write hive files? (y/n) [n]: e
 0  <SAM> - OK
```

### 4 - Reiniciar o servidor 

Comece por substituir o netboot por **Fazer boot no disco rígido** na Área de [Cliente OVHcloud](https://www.ovh.com/auth/?action=gotomanager) (ver [Etapa 1](./#1-reiniciar-o-servidor-em-modo-rescue_1)). 

De volta à linha de comandos, desmonte a partição e reinicie o servidor com os seguintes comandos:

```
# cd
# umount /mnt
# reboot

Broadcast mensagem from root@rescue.ovh.net on pts/0 (Wed 2020-05-27 11:28:53 CEST):

The system is going down for reboot NOW!
```

### 5 - definir uma nova password (IPMI)

Na Área de [Cliente OVHcloud](https://www.ovh.com/auth/?action=gotomanager), aceda ao separador `IPMI`{.action} para abrir uma sessão KVM.

![IPMI](images/adminpw_win_03.png){.thumbnail}

#### Etapa 5.1: para uma versão recente do Windows

A interface de ligação deve exibir uma mensagem indicando a expiração da palavra-passe.

![pwreset](images/adminpw_win_04.png){.thumbnail}

A nova palavra-passe do utilizador admin deve ser introduzida duas vezes. No entanto, o campo de confirmação ainda não está visível, o que significa que deve deixar o primeiro campo vazio, introduzir a nova palavra-passe no segundo campo e utilizar a tecla de tabulação ("↹") do teclado (virtual) para passar para o terceiro campo ("Confirmar a palavra-passe").
<br>Introduza novamente a password e clique na seta para a gravar.

![enterpw](images/adminpw_win_05.png){.thumbnail}

Clique em `OK`{.action} e estará ligado.

![adminlogin](images/adminpw_win_06.png){.thumbnail}

#### Etapa 5.2: para uma versão anterior do Windows

Uma janela da linha de comando (cmd) deve abrir-se quando a sessão KVM estiver estabelecida.

Defina a palavra-passe do utilizador atual ("Administrator"):

```
user Administrator *
```


![administratorpw](images/adminpw_win_07.png){.thumbnail}

> [!primary]
>
Recomenda-se a utilização do teclado virtual para introduzir palavras-passe nesta interface.
>

### Reinicialização da password admin com o auxílio de WinRescue

#### 1 - reiniciar o servidor em modo rescue

O sistema deve ser ativado em modo rescue antes de poder alterar a palavra-passe admin. Aceda à Área de [Cliente OVHcloud](https://www.ovh.com/auth/?action=gotomanager), aceda à secção `Bare Metal Cloud`{.action} e selecione o seu servidor na lista de navegação à esquerda em `Servidores dedicados`{.action}.

O netboot deve ser migrado para "WinRescue System for Windows". Procure "Boot" na zona **Informações gerais** e clique em `...`{.action} e depois em `Alterar`{.action}.
<br>Na janela que aparecer, selecione **Fazer boot em modo rescue** e selecione "WinRescue" no menu. Indique um endereço de e-mail no último campo se os dados de acesso tiverem de ser enviados para um endereço diferente do endereço principal da sua conta OVHcloud. 

Clique em `Seguinte`{.action} e depois em `Validar`{.action}.

![winrescuemode](images/adminpw_win_08.png){.thumbnail}

Concluída a alteração, clique em `...`{.action} à direita de "Status" na zona **Estado dos serviços**.
<br>Clique em `Reiniciar`{.action} e o servidor será reiniciado em modo rescue. Esta operação pode demorar alguns minutos.
<br>Pode verificar o progresso sob o separador `Tarefas`{.action}.
<br>Receberá um e-mail com os dados de acesso (incluindo a palavra-passe) do utilizador "root" do modo rescue.

![rescuereboot](images/adminpw_win_02.png){.thumbnail}

Para mais informações sobre o modo rescue, consulte [este guia](../ovh-rescue/).

#### Etapa 2: eliminar a password atual

Na Área de [Cliente OVHcloud](https://www.ovh.com/auth/?action=gotomanager), aceda ao separador `IPMI`{.action} para abrir uma sessão KVM.

![IPMI](images/adminpw_win_03.png){.thumbnail}

Para reinicializar as palavras-passe, a ferramenta NTPWEdit é necessária. Uma vez ligado através do KVM, abra o browser e descarregue-o a partir do [website oficial](http://www.cdslow.org.ru/en/ntpwedit/). 

Navegue até à pasta onde está o ficheiro ZIP descarregado e extraa o conteúdo. A seguir, abra o executável *ntpwedit64* para iniciar a aplicação.

![ntpwedit](images/adminpw_win_09.png){.thumbnail}

Nesta interface, pode manipular o ficheiro *SAM* para apagar a palavra-passe do utilizador admin. O caminho de acesso predefinido do diretório *WINDOWS* está pré-preenchido. Abra o ficheiro para apresentar a lista de utilizadores ao clicar em `Abrir`{.action}.

O utilizador em causa será "admin" ou "Administrator", consoante a versão do Windows. Se ambos estiverem presentes, escolha "admin". A seguir, clique em `Alterar palavra-passe`{.action}.

![ntpwedit](images/adminpw_win_10.png){.thumbnail}

Na nova janela, deixe os campos vazios e clique em `OK`{.action}. Clique em `Registar as modificações`{.action} e depois em `Sair`{.action}.

O servidor deve ser reiniciado.

#### Etapa 3: reiniciar o servidor 

Comece por substituir o netboot por **Fazer boot no disco rígido** na Área de [Cliente OVHcloud](https://www.ovh.com/auth/?action=gotomanager) (ver [Etapa 1](./#etape-1-redemarrer-le-serveur-en-mode-rescue_1)).

De volta à janela KVM, selecione a opção de paragem `Reiniciar`{.action} através do botão Windows "Iniciar" no canto inferior esquerdo.

Consulte este manual na [etapa 5: definir uma nova password (IPMI)](./#etape-5-definir-un-nouveau-mot-de-passe-ipmi).


## Quer saber mais?

[Ativar e utilizar o modo rescue](../rescue_mode/)

[Como utilizar o IPMI com servidores dedicados](../usar-ipmi-servidores-dedicados/)

Fale com a nossa comunidade de utilizadores: <https://community.ovh.com/en/>.
