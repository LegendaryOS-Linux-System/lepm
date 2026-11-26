# lepm — menedżer pakietów do budowy obrazów LegendaryOS

**Tylko** do budowy obrazów ISO/OCI (nie trafia do obrazu, nie jest używany na urządzeniu — tam jest **lpm**).

## Po co
Przy budowie obrazu nie ma jeszcze systemu, więc nie ma czym „atomowo instalować” pakietów, a LegendaryOS nie używa dnf
(Fedora Silverblue woła dnf w buildzie bootc — my mamy własne narzędzie). lepm składa rootfs od zera:

```
repos.hk → metadane rpm-md → satsolv (zależności, atrapy systemd) → pobranie + sha256 + GPG → bsdtar → sysusers → hooki → installed.xml
```

Cała logika jest w bibliotece **[lrpm](../lrpm)** (współdzielonej z lpm); lepm to cienkie CLI. Biblioteki **tylko przez bit**
(`use "bit -> lrpm" from "rp"`) — zero kopii źródeł. Napisany w **nula + Z# + H#** (część Z# → `mod/`, patrz niżej).

## Użycie
```
lepm bootstrap --root build/rootfs --repos repos.hk --policy compat.hk --cache build/cache \
      --release 43 --arch x86_64 --list package-list/base.list --list package-list/desktop.list
lepm plan  …              # to samo bez pobierania (dry-run)
lepm sync --repos repos.hk # sama synchronizacja metadanych
lepm search <fraza> --repos repos.hk
lepm doctor
```
Builder (`legendaryos-builder`) woła `lepm bootstrap` w kroku `packages`. Wymagane narzędzia: `curl gawk bsdtar zstd sha256sum chroot mount`
(+ `rpmkeys` przy `gpgcheck => true`).

## Rola Z#
`mod/names.zsharp` (walidacja nazw/celów) — szkic niezweryfikowany (brak kompilatora Z#, zob. ../Z-SHARP-GAPS.md Z1/Z3).
Walidacja faktycznie działa w H# (`lrpm::pure`).

## Czego lepm NIE robi
Nie jest atomowy, nie zna wdrożeń/rollbacku FastTree, nie uruchamia skryptletów RPM (zamiast tego: sysusers + hooki z `compat.hk`).
Zob. [../lrpm/docs/POLICY.md](../lrpm/docs/POLICY.md). **Status: niezweryfikowany** (brak kompilatora H#, libsolv, Fedory w sesji autora).

## 0.2.0 — hartowanie
- `lepm lists` — walidacja i rozwinięcie list pakietów bez sieci; brakujący plik `--list` jest błędem (wcześniej cicho pomijany).
- Blokada `<root>/.lepm.lock` z wykrywaniem martwego PID (kod wyjścia 2 gdy zajęte).
- `lepm clean` — czyści cache z kontrolą bezpiecznej ścieżki.

## 0.3.0 — pełny solver
`lepm solve [--installed installed.xml] [--erase S] [--update S] [--dist-upgrade|--update-all|--verify] [--opt O] [--pkg S|--list plik]`
— plan z satsolv (instalacja, wersje, globy, `provides:`, usuwanie z `cleandeps`, aktualizacje, konflikty z opisem).
`bootstrap`/`plan` przyjmują też `--erase` i `--opt` (np. `--erase systemd --opt no-weak`). Zob. lrpm/docs/SOLVER.md.
