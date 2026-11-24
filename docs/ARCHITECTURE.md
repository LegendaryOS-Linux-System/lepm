# lepm — architektura

* `src/main.h#` — parsowanie opcji, komendy `bootstrap|plan|sync|search|doctor|version`; wywołuje wyłącznie fasadę `rp::txn::*` z lrpm.
* `src/lists.h#` — wczytanie `package-list/*.list`, walidacja nazw, katalog docelowy.
* `mod/names.zsharp` — Z# (szkic, patrz README).
* Konwencje: importy bit tylko w `main.h#`; wywołania przez alias (`rp::txn::bootstrap`), do lrpm przekazujemy wyłącznie typy proste.

## Przepływ `bootstrap`
1. wczytaj listy, zwaliduj nazwy; 2. `txn::bootstrap` (lrpm): sync → solve (installed = ∅, atrapy compat) → fetch → rozpakowanie (`filesystem`, `setup` pierwsze) → sysusers → hooki → `usr/share/legendaryos/installed.xml`;
3. builder zamienia układ (`usr→system`…) — `installed.xml` jedzie w obrazie i jest bazą dla `lpm install`.

## Dlaczego osobne narzędzie, a nie lpm
lpm działa na gotowym, niezmiennym systemie (warstwy, composefs, rollback). lepm działa na pustym katalogu, jako root, w kontenerze
budowy — inne wymagania (bez atomowości, bez FastTree). Wspólny kod: lrpm.
