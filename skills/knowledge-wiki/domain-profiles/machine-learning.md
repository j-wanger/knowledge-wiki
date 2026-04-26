# Domain Profile: Machine Learning

Machine learning engineering, model development, MLOps, and AI systems. Frameworks and tooling evolve rapidly; foundational theory is stable.

## key_topics

- model-development
- training-infrastructure
- evaluation-and-testing
- deployment-and-serving
- data-engineering
- mlops

## conventions

- Version-pin all framework-specific advice (e.g., "PyTorch 2.x", "scikit-learn 1.4").
- Distinguish between framework-specific patterns (volatile) and mathematical foundations (stable).

## staleness_rules

```yaml
staleness_rules:
  default_days: 180
  overrides:
    - tags: [frameworks, tooling, libraries]
      days: 90
    - tags: [apis, cloud-services, pricing]
      days: 60
    - tags: [theory, mathematics, foundations]
      days: 365
```

**Rationale:** ML frameworks release breaking changes quarterly. Cloud API pricing and features shift frequently. Mathematical foundations (linear algebra, optimization theory) are evergreen.
