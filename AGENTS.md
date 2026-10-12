# AI Agent Guidelines & Architecture Rules

This document establishes binding conventions and architectural constraints for any AI agent or contributor developing in this `product`.

---

## 1. Core Principles & Hard Constraints

- **Zero External Dependencies**:
  Never introduce third-party package dependencies into `pyproject.toml` (o similar) dependencies or runtime code.

- **System Compatibility**:
  - Evaluar la mayor posible.

- **Architectural Separation**:
  - evaluar

---

## 2. Semantic Versioning Rules (SemVer)

Before committing, pushing, or preparing changes, the AI **MUST** evaluate the change type and bump version accordingly:

- **PATCH** (`0.x.Y` $\rightarrow$ `0.x.Y+1`):
  Bug fixes, documentation updates, test additions, and internal non-breaking refactors.

- **MINOR** (`0.X.0` $\rightarrow$ `0.X+1.0`):
  New features, new CLI commands, new flags, new protocol endpoints, or new client configuration support.

- **MAJOR** (`X.0.0` $\rightarrow$ `X+1.0.0`):
  Breaking changes to CLI subcommands/flags, public HTTP APIs, or configuration file formats.

### Version Synchronization Rule
The version string **MUST** always be updated simultaneously across both files:
1. `bridge/__init__.py`: `__version__ = "X.Y.Z"`
2. `pyproject.toml`: `version = "X.Y.Z"`

Never update one without the other.

---

## 3. Testing Discipline & Verification

- **Test-Driven Development (TDD)**:
  ...
- **Execution Command**:
  ...
- **100% Pass Rate**:
  All unit and integration tests must pass with zero failures and zero errors before any commit.
- **Test Count Synchronization**:
  Whenever tests are added or removed, update the test count stated in `README.md` to reflect the exact number of passing tests.

---

## 4. Git & Commit Guidelines

- **Conventional Commits**:
  All commit messages must adhere to conventional commits format:
  `feat(...)`, `fix(...)`, `refactor(...)`, `test(...)`, `docs(...)`, `chore(...)`.
- **No AI Attribution**:
  Never add `Co-Authored-By`, `Generated-by`, or any AI attribution tags to commit messages.

---

## 5. Developer Quick Reference

| Task | Command |
|---|---|
| Run full test suite | `python3 -m unittest discover -s tests -v` (u otro similar mas acorde) |
| Run specific test module | `python3 -m unittest tests/test_setup.py -v` (u otro similar mas acorde) |
| Test CLI entrypoint | `python3 -m bridge --help` (u otro similar mas acorde) |
| Check bridge version | `python3 -m bridge --version` (u otro similar mas acorde) |
| Run local smoke test | `python3 scripts/smoke_test.py --host 127.0.0.1 --port 24980` (u otro similar mas acorde) |
| Update in-place | `agy-bridge update` (or `python3 -m bridge update`) (u otro similar mas acorde) |
Evaluar otros posibles mas acorde al proyecto en si que estamos desarrollando.

## 6. Puntos a tener en cuenta si el producto lo amerita
1. Ocultar API Keys
2. Eliminar Secretos de Git
3. Key publica para BD
4. Row-Level Security
5. Encriptar datos
6. Forzar la autenticacion
7. Restringir el acceso a registros
8. Bloquear manipulacion de campos
9. Proteger  Session cookies
10. Hashear contraseñas
11. Rate limiting
12. Proteccion contra bots
13. Proteccion contra prompt injection
14. Parametrizar queries
15. Validar inputs
16. Sanitizar contenido
17. Restringir archivos
18. Devolver datos necesarios
19. Security headers
20. Forzar HTTPS
21. Escanear dependencias
