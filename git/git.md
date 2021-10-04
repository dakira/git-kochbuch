# Git Kochbuch
Eine Liste von unsortierten git Tipps.

## Inhalt

- [Grundbefehle](#grundbefehle)
- [Commits und ihre Nachrichten](#commits-und-ihre-nachrichten)
- [Workflow](#workflow)
- [Remotes](#remotes)
- [Branches](#branches)
    - [Neue Branches erstellen](#neue-branches-erstellen)
    - [Eine feature Branch in master mergen](#eine-feature-branch-in-master-mergen)
    - [Auf eine Branch wechseln, die Remote schon existiert](#auf-eine-branch-wechseln-die-remote-schon-existiert)
    - [Branches lokal und remote löschen](#branches-lokal-und-remote-löschen)
    - [Konflikte: Ours und Theirs](#konflikte-ours-und-theirs)
- [Undo](#undo)
    - [git revert (einen einzelnen Commit rückgängig machen)](#git-revert-einen-einzelnen-commit-rückgängig-machen)
    - [git reset (auf einen Commit zurückspringen)](#git-reset-auf-einen-commit-zurückspringen)
    - [git clean (nicht getrackte Dateien löschen)](#git-clean-nicht-getrackte-dateien-löschen)
    - [Undo als NEUER Commit AUF den ungewollten Commits](#undo-als-neuer-commit-auf-den-ungewollten-commits)
    - [Ungewollte commits in eigenen Branch schieben](#ungewollte-commits-in-eigenen-Branch-schieben)
- [Getrackte Dateien *vergessen*](#getrackte-dateien-vergessen)
- [Die Geschichte neu schreiben](#die-geschichte-neu-schreiben)
    - [Rebase to master](#rebase-to-master)
    - [Rebase Interaktiv](#rebase-interaktiv)
    - [Autoren eines einzelnen Commits ändern](#autoren-eines-einzelnen-commits-ändern)
- [Signierte Commits](#signierte-commits)
- [Diff](#diff)
- [Beispiele](#beispiele)
    - [Opensource Workflow](#opensource-workflow)

## Grundbefehle

|Befehl|Beschreibung|
|------|------------|
| `git init .` | In aktuellem Ordner ein Git Repository initialisieren |
| `git status` | Aktuellen Status des Repositories anzeigen |
| `git add <folders/files>` | Ausgewählte Ordner und Dateien stagen |
| `git add .` | Sämtliche Änderungen und neue Dateien stagen |
| `git commit -m '<message>'` | Neuer Commit mit angegebener Nachricht |
| `git push` | Änderungen an einen Server übertragen |
| `git pull` | Änderungen von einem Server übertragen |
| `git checkout <branchname>` | In einen anderen branch wechseln |

## Commits und ihre Nachrichten

Commits sollten möglichst atomar sein und nur Änderungen zu einem Thema enthalten. Die Commit-Nachrichten sollten grundsätzlich folgende Kriterien erfüllen:

- aussagekräftig
- auf Englisch formuliert
- im Präsenz gehalten

> **Tipp**: Eine gute Commit-Nachricht vervollständigt den folgenden Satz: *When applied this commit will...*

Beispiele:
- Update dependencies
- Implement AzureAD OIDC authentification
- Fix security-problem in controllers

Negativ-Beispiele:
- fix stuff
- my work from may
- deleted some files

## Workflow

Es gibt unterschiedliche Workflows, wie man im Team (oder allein) mit Git arbeiten kann. Nutzt man eine zentrale Verwaltung wie Github oder Gitlab, empfiehlt sich z.B. der s.g. [Github-Flow](https://guides.github.com/introduction/flow/). Dieser geht davon aus, dass in der master bzw main Branch immer der aktuellste lauffähige Code liegt. Jede Entwicklung findet in Feature- oder Bugfix-Branches statt.

1. Branch erstellen. Z.B. `fix-authentication-error` (s. [Branches](#branches))
2. Commits in dem Branch machen (und auf den server pushen)
3. Pull-Request (Github) bzw Merge-Request (Gitlab) erstellen
4. Im PR/MR Code-Review, Diskussion und Anpassungen
5. Code im PR/MR testen (CI/CD und/oder manuell)
6. PR in master/main mergen -> Deploy nach Production
7. Feature-/Bug-Branch löschen

Für ein konkreteres Beispiel s. [Opensource Workflow](#opensource-workflow)

Eine Erweiterung des Github-Flow ist der [Gitlab-Flow](https://docs.gitlab.com/ee/topics/gitlab_flow.html), welcher das Konzept um Environment-Branches erweitert (staging, production, usw.). Das ist dann sinnvoll, wenn aus Merges in bestimmte Branches ein Deployment getriggert werden soll.

## Remotes

Remotes sind *entfernte* Quellen für ein Repository. Es können zentrale Server sein, oder Versionen des Repositories von Kollegen im lokal Netz. Als einfachstes Beispiel kann man einen Klon von einem Github Repository nehmen. Da wird automatisch dieses Repository als **origin** Remote angelegt. Hierbei ist *origin* der konventionelle Name für das Repository von dem der eigene Code abstammt (und zu dem man i.d.R. seinen Code schickt). Eine weitere Konvention ist den Namen **upstream** für das Repository des Code-Besitzers zu hinterlegen.

> **Beispiel:** Man möchte an einem Opensource-Projekt mitarbeiten. Hierzu erstellt man einen Fork dieses Projektes, auf welchem man arbeitet. Dieser Fork ist aus der eigenen Sicht dann das *origin*-Remote wärend das eigentliche Repository des Projektes als *upstream*-Remote hinterlegt wird.

- Remotes anzeigen
    `git remote -v`
- Remote hinzufügen (z.B.)
    `git remote add origin git@server.com/user/project.git`  
    `git remote add upstream git@server.com/owner/project.git`
- Remote entfernen
    `git remote rm upstream`
    
## Branches

### Neue Branches erstellen

|Befehl|Beschreibung|
|------|------------|
| `git branch new-feature` | Erstellt die Branch `new-feature` |
| `git checkout -b new-feature` | Erstellt die Branch `new-feature` und wechselt hinein |

### Eine `feature` Branch in `master` mergen

Hat man ein Feature so weit, dass es in den Master-Zweig übernommen werden soll, muss man ein Merge durchführen. Dazu bringt man die Branch erst mal auf den aktuellen Stand von master (s. [Rebase to master](#rebase-to-master)), beseitigt etwaige Konflikte und führt erst dann den Merge durch.

Falls sich noch nicht commitete Änderungen im `feature` Zweig befinden, die nicht mit gemerged werden sollen, müssen diese voher mit `git stash` *versteckt* werden. Später können sie mit `git stash pop` wieder zurückgeholt werden. Gibt es Änderungen die einfach verworfen werden sollen, können diese mit `git restore <folder/files>` auf den letzten Commit zurückgesetzt werden.

Zuerst der Rebase: Auf der `feature` branch wird dazu folgender Befehl ausgeführt. `origin` ist hierbei das remote, welches die master-branch enthält in welche germerged werden soll.

`git pull --rebase origin master`

Treten nun [Konflikte](#konflikte-ours-und-theirs) auf müssen diese zuerst beseitigt werden. Dazu die Hinweise von `git status` befolgen. Nun folgt der Merge:

```bash
git checkout master
# noch mal schnell dafür sorgen, dass master auch wirklich auf dem Stand des remotes ist
git pull
git merge feature
# nun den merge auch noch auf den Server pushen
git push origin master
```

Es sollte ein fast-forward-Merge erfolgen. Die feature-branch kann nun [gelöscht](#branches-lokal-und-remote-löschen) werden.

### Auf eine Branch wechseln, die Remote schon existiert

> **Hinweis**: In aktuellen Git-Versionen wird bei einem checkout geprüft, ob eine Branch remote existiert und man muss nichts weiter machen als `git checkout <branch name>`

Das nennt sich im Git-Jargon auch *remote tracken*. Dazu gibt es bei git mehrere Wege, hier die kürzesten:

`git checkout --track <remote>/<remote_branch>`

oder

```bash
git fetch <remote> <remote_branch>:<local_branch> # z.B.
git fetch upstream develop:develop
```

Mit dem ersten Befehl wird die Branch vom remote geholt, eine gleichnamige lokale Branch erstellt, mit der remote Branch verbunden und zuletzt auf diese gewechselt. Der zweite Befehl macht das gleiche, aber ohne auf die Branch zu wechseln.

### Branches lokal und remote löschen

Braucht man eine Branch nicht mehr, weil ihr inhalt z.B. in master gemerged wurde, kann man sie löschen. Dies muss man mit zwei Befehlen für die lokale bzw. die remote Branch machen.

```bash
git branch -d <branch>    # löscht die Branch *nur* wenn der Inhalt gemerged wurde oder anderswo existiert
git branch -D <branch>    # löscht die Branch in jedem Fall
git push origin :<branch> # löscht die remote Branch
```

### Konflikte: Ours und Theirs

Bei unterschiedlichen Git-Aktionen wie Merge und Rebase kann es zu Konflikten kommen, wenn z.B. in beiden Versionen an der gleichen Datei gearbeitet wurde. Zum Auflösen der Konflikte können die betroffenen Dateien von Hand bearbeitet werden und die Konflikt-Kommentare entfernt werden, um dann den Merge oder Rebase fortzusetzen.

Manchmal will man aber auch einfach nur die eine oder die andere Version übernehmen. Z.B.

```bash
git checkout --ours -- path/to/some/dir
# oder
git checkout --theirs -- path/to/some/file
```

Aber was ist nun *ours* oder *theirs*, master oder feature? Die folgende Tabelle zeigt das für die möglichen Konfliktfälle in einer groben Vereinfachung.

| Operation | Ours | Theirs |
|-----------|------|--------|
| <pre lang="bash">git checkout master<br>git merge feature</pre> | master | feature |
| <pre lang="bash">git checkout feature<br>git rebase master</pre> | master | feature |
| <pre lang="bash">git checkout feature<br>git cherry-pick commit-id</pre> | feature | commit-id |
| <pre lang="bash">git checkout feature<br>git revert commit-id</pre> | feature | invers von commit-id |

*Ours* ist grob immer der HEAD der Branch, auf der die Operation ausgeführt wird. Bei einem Merge, die Branch, in welche hineingemerged wird. Bei Cherry-Picking ist es die Branch, auf welche ein einzelner Commit gesetzt werden soll.

Bei einem Rebase ist es komplizierter: Bei einem rebase auf master befindet man sich zwar auf einer Feature-Branch, die Operation wird aber *auf master* bzw dem HEAD-Commit von Master ausgeführt (und auch das ist noch eine Vereinfachung). Für weiterführende Erklärungen und Konfliktlösungsstrategien empfiehlt sich [Who is “us” and who is “them” according to Git?](https://stackoverflow.com/a/63911630/1501491) auf Stackoverflow.

## Undo

### git revert (einen einzelnen Commit rückgängig machen)

Mit `git revert <commit-id>` kann man die Änderungen eines beliebigen Commits aus der History rückgängig machen (und zwar nur dieses Commits, alle anderen Änderungen seit dem Commit bleiben bestehen).

### git reset (auf einen Commit zurückspringen)

Mit `git reset <commit-id>` setzt man die aktuelle Branch auf den Stand von `<commit-id>`. `git reset HEAD~` spring zum vorherigen Commit (*HEAD~*). Bei einem reset bleiben alle Änderungen der durch den reset gelöschen Commits als nicht-committete Änderungen bestehen.

Führt man stattdessen `git reset --hard <commit-id>` aus, sind auch die Änderungen weg.

### git clean (nicht getrackte Dateien löschen)

Mit `git clean -ndf` zeigt man sich alle Dateien und Ordner im aktuellen Verzeichnis an, die weder von git kontrolliert werden, noch in der *.gitignore* stehen. Das `-n` steht Unix-typisch für *dry-run*. Lässt man es weg werden die Dateien und Ordner tatsächlich gelöscht.

### Undo als NEUER Commit AUF den ungewollten Commits

Mit dem folgenden wird ein neuer Commit erstellt, der das Repository auf einen gewünschten alten Stand zurücksetzt. Dies ist nützlich bei einem öffentlich Repository, wo ein hard reset und force push nicht möglich ist.

```bash
# Reset the index and working tree to the desired tree
# Ensure you have no uncommitted changes that you want to keep
git reset --hard 56e05fced

# Move the branch pointer back to the previous HEAD
git reset --soft HEAD@{1}

git commit -m "Revert to 56e05fced"
```

### Ungewollte commits in eigenen Branch schieben

Manchmal hat man aus Versehen etwas auf dem `master` Branch commited, was eigentlich in einem Feature-Branch hätte landen sollen. Naïv könnte man annehmen man erstellt einfach ein Branch ausgehend von master und löscht danach die commits auf master (reset --hard).

Da der neue Branch master trackt und die Änderung (das Löschen der ungewollten commits) nach Erstellung des Branches erfolgten, würden die entspr. Commits bei einem rebase auf master kommentarlos gelöscht werden.

So funktioniert es:

```bash
git reset --keep <commit-id>
git checkout -t -b <branch-name>
git cherry-pick ..HEAD@{2}
```

Zuerst löschen wir die Commits auf dem master Branch in dem wir den Branch auf (`--keep` ist wie `--hard` aber mit Schutz vor Datenverlust). Danach erstellen wir den Branch auf dem die Commits zukünftig landen sollen. Mit `cherry-pick` holen wir uns nun einzeln alle Commits aus dem reflog zurück. `HEAD@{2}` entspricht hierbei dem Stand des Repositories vor 2 Aktionen, also vor dem Erstellen des Branches und vor dem reset.

## Getrackte Dateien *vergessen*

Wenn Dateien und Ordner unter git-Kontrolle stehen und man möchte Änderungen ignorieren gibt es drei Möglichkeiten:

1. Man behält die lokale Datei aber sie wird aus dem Repository entfernt.
   
    `git rm --cached <datei>` oder  
    `git rm -r --cached <ordner>`

2. Für Optimierung von großen Ordnern mit vielen Dateien (z.B. SDKs), die nie lokal verändert werden. Teilt git mit, dass es im genannten Ordner nicht nach Änderungen suchen braucht, da es keine geben wird.
   
    `git update-index --assume-unchanged <pfad>`

3. Sagt git, dass man seine eigene abweichende Version einer Datei oder eines Ordners behalten will. Praktisch für config files die sich und production/staging unterscheiden.
   
    `git update-index --skip-worktree <path-name>`
   
    Es ist wichtig zu wissen, dass Einstellungen, die man per `git update-index` **immer nur lokal** gesetzt werden. D.h. jeder Nutzer muss diese für sich selbst neu setzen.

## Die Geschichte neu schreiben

So lang man Commits nur lokal getätigt hat und nicht auf einen Server gepusht hat, kann man sie beliebig bearbeiten bzw. aufräumen. Das Tool dafür ist `git rebase`. Die beiden meistgenutzten rebases sind ein rebase auf einen anderen Zweig (i.d.R. master) und ein interaktives rebase zum beliebigen bearbeiten der History.

### Rebase to master

Gerade bei Opensource-Projekten wird man häufig gebeten einen Pull-Request auf master zu basieren (*rebase to master*). Zur Veranschaulichung stelle man sich vor man hat einen Dev-Zweig vom Master-Zweig bei Commit C gebrancht und auf diesem Dev-Zweig zwei Commits getätigt, die nun per Pull-Request wieder ein den Master-Zweig gemerged werden sollen.
```
master  A-B-C
             \
dev   (A-B-C)-A1-B1
```

Dummerweise hat es nun auf dem Master-Zweig zwischendrin einige Veränderungen gegeben.
```
master  A-B-C-D-E
             \
dev          A1-B1
```

Ein Merge unserer zwei Commits würde dadurch tatsächlich zu drei Commits führen. Einem *"Merge-Commit"*, bei dem etwaige Komflikte behoben werden (oder auch nicht, falls unnötig) und den beiden tatsächlichen Commits. Die meisten Maintainer bevorzugen s.g. *fast-forward* Merges, bei denen nur die getätigten Commits an den Master-Zeig angehängt werden und ein Merge-Commit nicht nötig ist. Dies erreicht man in unserem Fall über ein *rebase to master*. Dazu wird unser Zweig auf den Stand von master gesetzt (Commit E). Dann werden per Cherry-Picking die Commits A1 und B1 wieder eingespielt. Kommt es zu Konflikten hält das rebase an, wirft einen in eine Shell und erlaubt einem (unter Anleitung) die Konflikte zu beheben. Das Resultat sieht so aus:

```
master  A-B-C-D-E
                 \
dev              A1-B1
```
Und nach einem Merge so:
```
master  A-B-C-D-E-A1-B1
```
Das ganze klingt kompliziert, ist in der Praxis allerdings recht einfach. Will man bei einem Opensource-Projekt eine feature-branch auf den Master-Zweig des Upstream-Repositories rebasen geht das z.B. so:
```
git pull --rebase upstream master
```

Geht es um dem Master-Zweig des eigenen Projektes (weil z.B. Kollegen Änderungen an master durchgeführt haben) ersetzt man upstream durch origin (s. [Remotes](#remotes) weiter oben). Das obige ist die Kurzform des folgenden:
```
git fetch upstream
git rebase upstream/master
```

### Rebase Interaktiv

Für ein interaktives Rebase sucht man sich zuerst die Commit-ID des letzten Commits *vor* dem, den man noch mit bearbeiten möchte. Hat man insgesamt die Commits A, B, C, D, E und möchte D und E bearbeiten ist dies der Commit C. Man startet das interaktive Rebase über:
```
git rebase -i <commit-id>
```
Git zeigt einem nun im Standard-Editor alle zu bearbeitenden Commits an. Diese kann man beliebig umsortieren (so lang sie nicht voneinander abhängen) und zusätzlich folgende Veränderungen an ihnen durchführen:

- p, pick = Commit unverändert verwenden
- r, reword = Commit verwenden, aber Commit-Beschreibung bearbeiten
- e, edit = Commit verwenden, aber zum Nachbessern anhalten (das Arbeitsverzeichnis wird dazu auf den Stand dieses Commits gesetzt und man kann beliebig Änderungen durchführen)
- s, squash = Commit verwenden, aber mit vorherigem Commit vereinen
- f, fixup = wie "squash", aber diese Commit-Beschreibung verwerfen
- d, drop = Commit entfernen

### Autoren eines einzelnen Commits ändern

Ein spezieller Anwendungsfall des rebase. Man markiert den entsprechenden Commit für ein edit und landet in der Shell. Dort kann mann mit
```
git commit --amend --no-edit --author="John Doe <doe@example.com>"
```
den Autoren neu setzen.

## Signierte Commits

Man kann Commits mit GPG signieren:
```
git commit -S
```
Will man bei einem rebase die Signaturen behalten geht das so:
```
git rebase -i --gpg-sign=myemail@example.com <commit-id>
```
## Diff

Unterschiede anzeigen:

- Zwischen dem aktuellen Arbeitsverzeichnis und dem letzten Commit.  
    `git diff`
- Zwischen dem aktuellen und dem letzten Commit.
    `git diff HEAD~ HEAD` oder  
    `git show`
- Zwischen zwei beliebigen Commits.
    `git diff <commit_id1> <commit_id2>`
- Mit einem grafischen Programm
    `git difftool`

## Beispiele
### Opensource Workflow

Hier in Kürze, wie man zu einem Opensource-Projekt beiträgt:

1. **Forken**
    Auf bspw. Github kann man durch einen Fork eine Kopie des Projektes in seinem privaten Namespace erstellen. Auf dieser Kopie wird gearbeitet. Dazu klickt man i.d.R. im Webinterface einen Fork-Button.
2. **Klonen und Remotes**
    `git clone git@github.com:USERNAME/PROJECTNAME.git`  
    Nun hinterlegt man noch die Original-Quelle als *remote*.  
    `git remote add upstream git@github.com:PROJECT_USERNAME/PROJECTNAME.git`
3. Branchen
    Für den zu schreibenden Code erstellt man nun einen passenden Branch wie *fix-issue123* oder *add-contact-form*  
    `git checkout -b BRANCH_NAME`
4. **hack hack hack**
    Nun arbeitet man am Code. Arbeitet man länger an einem Feature, lohnt es sich gelegentlich Änderung der Master-Branch des Upstream-Projekts zu mergen ([rebase](https://git-scm.com/book/en/v2/Git-Branching-Rebasing) auf master).  
    `git pull --rebase upstream master`
5. **Pull-Request**
    Die fertige (oder auch unfertige) Arbeit pusht man nun in den eigenen Namespace und ertellt einen Pull-Request. Zuerst muss der Code gepusht werden.  
    `git push origin BRANCH_NAME`  
    Nun kann man im Web-Interface einen PR erstellen.
6. **Warten und fixen**
    In der Regel wünschen sich Maintainer noch Änderungen. Um den PR zu verändern muss man einfach nur im gleichen Branch commiten und pushen. Oft verlangt der Maintainer noch mal einen [rebase to master](#rebase-to-master) (s.o.).
7. **Aufräumen**
    Wenn der Pull-Request angenommen (oder abgelehnt) wurde, kann man die lokale und die remote Branch löschen:  
    ```
    git checkout master
    git branch -D BRANCH_NAME
    git push origin --delete BRANCH_NAME
    ```
