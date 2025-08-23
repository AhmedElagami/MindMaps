# Admission Controllers

## Content

- Intercept API requests before persistence.
- Include built-in plugins and webhooks.

### ValidatingAdmissionPolicy
- Uses CEL expressions to validate requests without webhooks.
- Defined as cluster-scoped resources paired with `ValidatingAdmissionPolicyBinding`.
