# Core Concepts

## AI-Native Intermediate Representation (AIR)

The AI-Native Intermediate Representation is the foundational layer of Project Emet. Unlike traditional intermediate representations (IR) that serve as a bridge between human-readable source and machine code, AIR is designed as the primary representation.

### Key Characteristics

- **Self-Describing**: AIR contains rich semantic information that makes it comprehensible to AI systems
- **Verifiable by Design**: Every construct in AIR can be formally verified
- **Language-Agnostic**: AIR is not tied to any particular source language
- **Optimized for Reasoning**: Structure optimized for automated reasoning and proof systems

## Human-Readable Code as Legacy Artifact

Traditional programming languages serve as legacy interfaces to the AIR. In the Emet paradigm:

- Source code is **generated from AIR**, not the other way around
- Multiple source representations can exist for the same AIR
- Source languages are projection layers, not the source of truth
- Human readability is a feature of the projection, not the core

## Verification-First Architecture

Project Emet treats verification as a first-class concern, not an afterthought:

- **Intrinsic Correctness**: Properties are embedded in the AIR itself
- **Formal Semantics**: Every operation has a precise mathematical meaning
- **Proof Objects**: Verification produces proof objects that travel with the code
- **Automated Reasoning**: AI systems can automatically verify and optimize

## The Emet Compilation Model

Traditional: `Source → IR → Machine Code`

Emet: `AIR ⇄ AI Systems → (Human-Readable Projections | Machine Code)`

Where:
- AIR is the source of truth
- AI systems manipulate and verify AIR directly
- Human-readable code is generated on-demand
- Machine code is one of many possible targets

## Design Philosophy

### Inverting the Traditional Model

Instead of compiling from human-readable to machine-readable:
1. Express computational intent in AIR
2. Let AI systems verify and optimize
3. Generate human-readable views when needed
4. Compile to various targets (including traditional languages)

### Embracing Machine Intelligence

Project Emet acknowledges that:
- AI systems can reason about code more effectively than humans
- Formal verification should be the default, not the exception
- Human-centric syntax is often a barrier to correctness
- The future of programming is collaborative human-AI development

## Application Domains

Project Emet is particularly suited for:

- **Safety-Critical Systems**: Where formal verification is essential
- **AI-Generated Code**: Where human readability is secondary
- **Cross-Language Interoperability**: Where AIR serves as a universal representation
- **Automated Reasoning**: Where machines need to understand and manipulate code
- **Future Programming Paradigms**: Where the line between human and AI authorship blurs
