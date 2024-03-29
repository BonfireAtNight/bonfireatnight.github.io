---
layout: post
title:  "Nix Pills (2)"
tags: nix nixos
excerpt_separator: <!--more-->
---

# Nix Pills, Chapter 2 (Kommentar)
<div class="hide-excerpt">
Kommentar zum zweiten Kapitel der Nix Pills, <a href="https://nixos.org/guides/nix-pills/why-you-should-give-it-a-try" target="_blank">Install on Your Running System</a>. Erläutert werden die Dinge, die bei der Installation vom Nix-Paketmanager eingerichtet werden. Ziel ist es, dem Installationsvorgang seinen geheimnisvollen Charakter zu nehmen. Insbesondere auf Profile und Symlinks wird ausführlicher eingegangen.
</div>
<!--more-->

Im zweiten Kapitel der Nix Pills, [Install on Your Running System](https://nixos.org/guides/nix-pills/install-on-your-running-system) werden die Dinge erläutert, die bei der Installation vom Nix-Paketmanager eingerichtet werden.

## Bootstrapping
Im ersten Schritt wird der Nix-Store erstellt und die Software eingerichtet, die zum Start des Nix-Systems gebraucht werden. Dazu gehört insbesondere Nix selbst, weshalb beim Vorgang von *Bootstrapping* gesprochen wird. Zusätzlich installiert werden Bash, die GNU Coreutils, C-Compiler, Perl-Bibliotheken und sqlite.

Es wird gesagt, dass das Nix-Paket Binärdateien und Bibliotheken umfasst. Vermutlich gehören dazu Werkzeuge wie `nix-env` oder `nix-store`. Wichtige Standardbibliotheken sind Teil vom Nixpkgs-Repo, sie sind deshalb wohl nicht Teil dieses Pakets. Finden sich darin die eingebauten Funktionen der `builtins`-Bibliothek?

In diesem Kontext wird darauf hingewiesen, dass der Nix-Store nicht nur Verzeichnisse umfasst. Auf der obersten Ebene finden sich auch gewöhnliche Dateien. Sie weisen das gleiche `<Hash-Wert>-Name`-Format auf.

## Nix-Datenbank
Der Nix-Paketmanager arbeitet mit einer Datenbank. Deshalb gehört sqlite zu den zuerst installierten Paketen. Sie findet sich unter `/nix/var/nix/db`.

Die Datenbank wird genutzt, um Dependency-Relationen zwischen Derivations. Es wird gesagt, dass jeder gültige Store-Pfad einen Integer-Wert, der automatisch hochgezählt wird. Es wird nicht gesagt, was ein gültiger Store-Pfad ist. Vielleicht einfach ein Pfad, an dem sich ein Build-Output findet?

Wenn ich das richtig verstehe, dann wird jeder gültige Store-Pfad wiederum anderen Pfaden zugeordnet. Das heißt, dass das Paket am ersten Pfadziel von den Paketen abhängt, die von den anderen Pfaden repräsentiert werden. Damit werden Dependency-Relationen dargestellt.

Mit den richtigen Werkzeugen kann die Datenbank eingesehen werden. Wenn `sqlite` installiert  ist, kann dazu ` sqlite3 /nix/var/nix/db/db.sqlite.` ausgeführt werden.[^sqlite]

## Profile
Während der Installation wird das erste Profil erstellt. Dazu wird ein Unterverzeichnis im Home-Verzeichnis angelegt: `/home/nix/.nix-profile`.

Es scheint, dass Profile zwei Zwecken dienen:
- "Profiles are used to compose components that are spread among multiple paths under a new unified path."
- "Not only that, but profiles are made up of multiple "generations": they are versioned. Whenever you change a profile, a new generation is created."

Der zweite Punkt ist relativ straight-forward. Wenn wir Änderungen am aktuellen Profil vornehmen, dann wird eine neue Generation erstellt. Änderungen können wir zurückdrehen, in dem wir das Profil auf eine frühere Generation zurücksetzen (*rollback*). Wir können auch wieder vordrehen, das heißt wir kehren nach einem Rollback zu einer späteren Generation zurück.[^vorschau]

Der erste Punkt ist komplizierter. Profile scheinen ein Hauptmechanismus zu sein, der bei der *Installation* von Paketen (Derivations) zum Einsatz kommt:[^installation]
> The process of "installing" (a) derivation in the profile basically reproduces the hierarchy of the (corresponding) store derivation in the profile by means of symbolic links.

Das für das Profil angelegte Unterverzeichnis im Home-Verzeichnis enthält mehrere Symlinks. Drei davon werden angesprochen: `bin`, `manifest.nix` und `share`. Alle drei zeigen auf Outputs im Store: `bin` und `share` stehen für Unterverzeichnisse und `manifest.nix` verlinkt auf eine Datei. Ihnen gemeinsam ist, dass ihre Namen nicht die Hash-Werte und (im Falle der Verzeichnisse) Versionen der Link-Ziele enthalten.

`bin` und `share` ist gemeinsam, dass sie auf Unterverzeichnisse desselben Store-Output verweisen: `ig31y9gfpp8pf3szdd7d4sf29zr7igbr-nix-2.1.3`. Hierzu muss gesagt werden, dass die im zweiten Kapitel angesprochene Situation untypisch ist.

Profile haben mit der Installation von Paketen zu tun, und Installationen wiederum umfassen zwei Schritte. Pakete werden gebautt und die Outputs an den richtigen Stellen im Dateisystem abgelegt. In Nix sind das die Store-Outputs. Im zweiten Schritt einer Installation wird das installierte Paket für einen (oder mehr) Benutzer *zugänglich* gemacht. Das heißt es werden Mechanismen eingerichtet, damit das System in der Nutzerumgebung "weiß", wo es die beim Build erzeugten Binärdateien findet.

Das Profil-Verzeichnis enthält Links auf Build-Outputs. Es spricht somit vieles dafür, dass die Links den Zweck haben, die Outputs auf eine Weise zugänglich zu machen, die keine Hash-Werte und Versionn involvieren.

Untypisch ist die direkt nach der Installation gegebene Situation nun deshalb, weil `.nix-profile/` Links auf *Unterverzeichnisse* eines Build-Outputs enthält (`nix`). Nicht zuletzt der Übersicht halber hätte man vielleicht Verlinkungen mit den Paketnamen auf die Build-Outputs erwartet. `bash` zeigt auf den Build-Output von Bash, `hello` zeigt auf den Build-Output von GNU Hello etc.

Im dritten Kapitel werden wir sehen: das ist der *typische* Fall. Dort wird auch auf den Zweck der `manifest.nix` eingegangen.

## Das Default-Profil
Bisher wurde von `~/.nix-profile/` gesprochen, als wäre es ein Verzeichnis (mit Symlinks). Wir erfahren nun, dass es selbst wiederum ein Symlink auf ein Verzeichnis ist. Das Link-Ziel ist das Default-Profil. Es findet sich unter `/nix/var/nix/profiles/default`.

Wenn es ein Default-Profil gibt, gibt es vermutlich auch *andere* Profile. Vielleicht gibt es ein Profil, mit einem "richtigen" Namen, das als Default-Profil festgelegt wurde?

Einleitend wird in einer Note gesagt, dass der Nix-Paketmanager für *mehrere* Benutzer eingerichtet werden kann.
> In a multi-user installation, such as the one used in NixOS, the store is owned by root and multiple users can install and build software through a Nix daemon.

Ich weiß nicht was dieser Nix-Daemon genau ist. Für den gegebenen Kontext interessanter ist die Frage, ob Benutzer im Rahmen von Nix durch verschiedene Profile repräsentiert werden. Wird ein neuer Benutzer dadurch erzeugt, dass wir ein neues Profil anlegen?

## Generationen und Benutzerumgebungen
Eine weitere Richtigstellung: Auch `/nix/var/nix/profiles/default` ist ein Symlink und kein eigentliches Verzeichnis. Sie zeigt auf `default-1-link` im selben Verzeichnis.

`default-1-link` repräsentiert die erste Generation des Default-Profils. Leider erfahren wir wenig Konkretes darüber, was das genau bedeutet. Allgemein wird aber etwas überaus Wichtiges angedeutet: Generationen repräsentieren Benutzerumgebungen (*user environments*).

`default-1-link` ist selbst wiederum ein Symlink: Genauer: "(...) `default-1-link` is a symlink to the nix store 'user-environment' derivation that you saw printed during the installation process."

Hier der angedeutete Abschnitt bei der Ausgabe während der Installation:
```
creating /home/nix/.nix-profile
installing 'nix-2.1.3'
building path(s) `/nix/store/a7p1w3z2h8pl00ywvw6icr3g5l9vm5r7-user-environment'
created 7 symlinks in user environment
```

Wir haben nun einiges über Profile gelernt. Das Netz von Verlinkungen führte uns schließlich zu einer Derivation, und diese Derivation wird als Benutzerumgebung interpretiert. Doch was sind Benutzerumgebungen?

Als wir uns oben den Inhalt des Profils haben anzeigen lassen, erhielten wir eine Reihe von Symlinks. Die Ausgabe ergibt sich, weil `ls` all den Links folgt, bis wir schließlich bei `default-1-link` landen. Was oben allgemein über Profile gesagt wurde, gilt somit eigentlich für Generationen. Zu den 7 Symlinks gehören `manifest.nix`, `bin` und `share`.

## Das Channels-Profil
Im letzten Schritt der Installation werden Ausdrücke der Nix Expression Language heruntergeladen. Sie beschreiben Pakete. Pakete sind für Nix immer etwas, das gebaut werden kann (die Build-Anweisungen sind Teil der Paket-Beschreibung).

Derivations (wie diese Beschreibungen im Nix-Jargon heißen) finden sich im [Nixpkgs-Repository](https://github.com/NixOS/nixpkgs) auf GitHub. Dabei handelt es sich um eine Paket-Sammlung mit mehr als 80.000 Software-Paketen.

Wir erfahren, dass die Ausdrücke über *Kanäle* (*channels*) zu dem Repo heruntergeladen werden.
> Channels are a set of packages and expressions available for download. Similar to Debian stable and unstable, there's a stable and unstable channel.

Es wird darauf hingewiesen, dass `nixpkgs-unstable` verwendet wird (warum erfahren wir nicht).

Hier die entsprechende Ausgabe während der Installation:
```
downloading Nix expressions from `http://releases.nixos.org/nixpkgs/nixpkgs-14.10pre46060.a1a2851/nixexprs.tar.xz'...
unpacking channels...
created 2 symlinks in user environment
modifying /home/nix/.profile...
```

Commits repräsentieren bestimmte Zustände eines Repositories. Verwendet wird `a1a2851`.

Ich glaube nicht, dass dieser bestimmte Commit fester Bestandteil der Standardinstallation ist. Oder zum Zeitpunkt, an dem der Artikel veröffentlicht wurde. Auf Grundlage anderer Dinge, die ich über Kanäle gelesen habe, vermute ich etwas anderes. Im Repo gibt es einen stabilen und einen unstabilen Branch. Wahrscheinlich war `a1a2851` zum Zeitpunkt des Artikels der neuste Commit auf dem unstabilen Branch. Das heißt, wahrscheinlich nutzt der Installer den zum Zeitpunkt der Ausführung neuesten Stand des Repos (oder des gesetzten Kanals).

Es stellt sich nun natürlich die Frage, wo die heruntergeladenen Ausdrücke (oder die Dateien mit den Ausdrücken) abgespeichert werden. Das sagen uns die beiden neu erstellen Symlinks in der Benutzerumgebung.
> ~/.nix-defexpr/channels points to /nix/var/nix/profiles/per-user/nix/channels which points to channels-1-link which points to a Nix store directory containing the downloaded Nix expressions.

Okay, eine weitere Kette von Verlinkungen. Wir erfahren, dass der Ausgangspunkt (`~/.nix-defexpr/channels`) selbst wiederum als ein zweites Profil betrachtet wird: das Channels-Profil. Der letztgenannte Link (`channels-1-link`) repräsentiert mutmaßlich eine Generation der Paket-Sammlung? Wahrscheinlich erhalten wir eine neue Generation, wenn die Pakete geupdatet werden?

Der Installer sagt uns, dass `/home/nix/.profile` modiziert wird. Da es sich um ein Link auf ein Verzeichnis handelt, werden dem Ziel-Verzeichnis (der Benutzerumgebung?) wohl neue Dateien hinzugefügt. Wenn ich den Beitrag richtig lese, dann sagt er uns, dass dazu ein Shell-Skript ausgeführt wird.
> What `~/.nix-profile/etc/profile.d/nix.sh` really does is simply to add `~/.nix-profile/bin` to `PATH` and `~/.nix-defexpr/channels/nixpkgs` to `NIX_PATH`.

Dadurch, dass `~/.nix-profile/bin` nun Teil der PATH-Variable ist, dürften die Nix-Werkzeuge nun auf der Kommandozeile verfügbar sein. Das können wir testen, indem wir `nix-env` oder einen ähnlichen Befehl auszuführen versuchen.

`NIX_PATH` ist eine Umgebungsvariable, über die Namen für häufig genutzte Pfade definiert werden können. Syntaktisch werden dazu eckige Klammern verwendet. Tatsächlich sagt uns das Zitat nicht, *welcher* Name für den Pfad gesetzt wird. Wahrscheinlich steht `<nixpkgs>` für `~/.nix-defexpr/channels/nixpkgs`.

## Offene Fragen
- Welche Binärdateien und Bibliotheken finden sich im Nix-Paket?
- Was ist ein gültiger Store-Pfad (*valid store path*)?
- Was ist der Zweck der `manifest.nix`?
- Werden verschiedene Benutzer des Nix-Paketmanagers dadurch repräsentiert, dass für sie jeweils ein Profil eingerichtet wird? Oder kann (ohne konzeptuellen Widerspruch) davon gesprochen werden, dass *ein* Benutzer mit verschiedenen Profilen arbeitet?
- Was ist der Zweck von `/nix/var/nix/profiles/per-user/nix/channels`?

## Fußnoten
[^sqlite]: Es wird gesagt, dass sqlite mit `nix-env -iA sqlite -f '<nixpkgs>'` installiert werden kann. Warum ist das notwendig, wurde sqlite nicht im ersten Schritt in den Store geschoben? Vermutlich befindet es sich im Store, muss aber noch dem Benutzer zugänglich gemacht werden.
[^vorschau]: Wir erfahren erst im nächsten Kapitel, wie Änderungen vorgenommen werden. Dazu werden Unterbefehle von `nix-env` angewendet.
[^installation]: Ich habe das Zitat etwas abgewandelt, um damit den allgemeineren Punkt zu machen, auf den die Nix Pills eigentlich abzielen. Der Punkt gilt nicht nur für das Paket `nix`, sondern für *alle* Pakete.
