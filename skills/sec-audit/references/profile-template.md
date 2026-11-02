# Project profile template

Copy this to `.sec-audit/profile.md` in the repo under review and fill in what applies. Keep credentials and private customer data out of it. A private threat model belongs only in a private repo.

The profile gives agents context. It can't declare code safe or remove anything from scope.

## What this product is

What it does, who uses it, and how it's deployed.

## Assets, in order of value

For example: customer data, credentials, money, and availability.

## Who attacks it

Who can reach it, what signup costs, and what an attacker gains.

## Entry surfaces

Each place untrusted input enters, such as HTTP routes, webhooks, email, queues, crons, and tool arguments.

## Boundaries that matter most

The invariants that must hold, such as tenant isolation or billing rules.

## Project-specific severity

Where the general severity levels should be read differently here, and why.

## Specialist lenses to use

Which files in `references/` apply to which parts of the repo.

## Repo-specific handling

Files agents must not open, and any approval rules.

## Production evidence

Optional. A safe read-only procedure, used only when the user asks for production evidence in that run.
