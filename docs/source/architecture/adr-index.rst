.. _adr-index:

Architecture Decision Records (ADRs)
=====================================

An Architecture Decision Record (ADR) documents a significant architectural choice, the context
that led to it, and the consequences of the decision.

ADR Format
----------

Each ADR follows the shared template. To create a new ADR:

1. Copy ``adr/ADR-NNN-template.md`` to ``adr/ADR-NNN-<kebab-case-title>.md``
2. Fill in all sections
3. Add a row to the index table below
4. Open a PR – ADRs are reviewed like code

.. toctree::
   :hidden:
   :caption: ADR Files

   adr/ADR-NNN-template

Template
--------

* :doc:`adr/ADR-NNN-template` — Blank template (copy this to start a new ADR)

ADR Index
---------

.. list-table::
   :header-rows: 1
   :widths: 15 55 30

   * - ADR
     - Title
     - Status
   * - :doc:`adr/ADR-NNN-template`
     - Template
     - ✅ Accepted
