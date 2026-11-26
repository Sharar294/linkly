# ADR-0001: Use Python and FastAPI for the API and worker

- Status: Accepted
- Date: 2026-11-26

## Context

I'm learning a lot of infrastructure tools at once, so I need a language I'm proficient in. 

This project will follow a spec-first approach for the API contract.

The API will be redirect heavy so most of the request time is spent on Redis and Postgres. 

## Decision

Python 3.12, FastAPI, Pydantic, SQLAlchemy 2.0 (async), uv.

## Alternatives considered

I considered using Go but chose Python for familiarity and higher proficiency.

I chose FastAPI over Django and Flask due to simplicity and it's built-in functionality.

## Consequences

Python is slower than Go. 

Since FastAPI generates a spec from the code and I will write one by hand, CI has to 
ensure they match.
