# INTENT: nibble

> Takes just a nibble to decide: fast, typed choices for agent pipelines, built on Kev, plus a trainer for small task-specific deciders.

Status: placeholder name · written 2026-10-17 from David's direction · **this file changes only when David agrees.**

## What it is
- A decision layer: given a state and typed questions (choose one / choose none / score), returns calibrated probabilities. No text generation.
- **Built on Kev** (jaredpalmer/kev, Apache-2.0; local clone `~/Projects/vendor/kev`) as a dependency, with clear attribution.
- A **trainer** that turns logged decisions (from bonfire runs) into small task-specific deciders that run on a 3060 or CPU.
- First domain: game generation (which template, which primitives, which asset style, is this playable).

## What it is not
- Not a from-scratch reimplementation of Jev (decided 2026-10-17: build on Kev).
- Not a coding-workflow labeler (what system-a drifted into).
- Not a chat model or a general LLM.
- Not vendoring/copying Kev's code: depend on it, contribute upstream where sensible.

## The proving demo
In playdough's pipeline, nibble picks templates and primitives for a prompt **faster and at least as well as asking the big model**, measured on a held-out set of prompts.

## Interface to the stack
Input: state text + questions (JSON). Output: decision record (probabilities, pick, latency). Training input: bonfire `runs/*/decisions.jsonl`.

## Starting point
Kev as a dependency + lessons from system-one-mini (see docs/salvage.md).

## Open decisions
- Which Kev size to start with (0.8B / 4B / 9B) on a shared 3060.
- Decision suite format for game generation (draft once playdough has a first pipeline).
- Whether Soup (MakazhanAlpamys/Soup) is the trainer for small deciders.
