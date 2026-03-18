# LucidLabsAU — Copilot Instructions

Default instructions for all LucidLabsAU repositories. Project-level `.github/copilot-instructions.md` files override these defaults.

## Language

- Use Australian English spelling (organisation, colour, behaviour, licence, centre, etc.)
- Technical terms and code identifiers remain in their original form

## Coding Standards

- Write clean, readable code with meaningful names
- Prefer explicit types over `any` or implicit types
- Handle errors at boundaries — don't swallow errors silently
- Keep functions focused and single-purpose
- Organise imports logically (built-ins → external → internal → relative)

## Security

- **Never** commit secrets, API keys, tokens, or credentials
- Use `DefaultAzureCredential` for Azure authentication — never connection strings or API keys
- Store secrets in Azure Key Vault, never in environment variables or code
- Use parameterised queries — never string interpolation for SQL
- Validate all user input at system boundaries
- Add `.env` files to `.gitignore`

## Dependencies

- Prefer built-in features over new dependencies
- Document why each dependency is needed
- Keep dependencies up to date via Dependabot
- Audit for vulnerabilities before merging

## Testing

- Write tests for business logic and critical paths
- Use descriptive test names that explain the expected behaviour
- Follow Arrange-Act-Assert pattern
- Prefer integration tests over extensive mocking

## Git

- Write clear, concise commit messages focused on "why" not "what"
- Never force-push to main/master
- Keep PRs focused — one concern per PR
- Include a test plan in PR descriptions
