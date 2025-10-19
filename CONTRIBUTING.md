# Contributing to Navigator

Thank you for your interest in contributing to Navigator! This document provides guidelines for contributing to the project, especially when implementing features from the [NEXT_STEPS.md](NEXT_STEPS.md) roadmap.

## Table of Contents

- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Code Standards](#code-standards)
- [Testing Requirements](#testing-requirements)
- [Submitting Changes](#submitting-changes)
- [Implementing Roadmap Features](#implementing-roadmap-features)

## Getting Started

### Prerequisites

- Elixir 1.18.4
- Erlang/OTP 27.0.1
- Node.js 22.x
- PostgreSQL 13+

### Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/bruhdev1290/securityhub.git
   cd securityhub
   ```

2. **Install dependencies and setup database:**
   ```bash
   make setup
   ```
   Note: This can take 10-25 minutes on first run.

3. **Start the development server:**
   ```bash
   make dev
   ```
   The application will be available at `http://localhost:4000`

### Using Docker

Alternatively, you can use Docker Compose:

```bash
docker compose up
```

For LLM functionality, provide your OpenAI API key:

```bash
OPENAI_API_KEY=sk-proj... docker compose up
```

## Development Workflow

### 1. Create a Feature Branch

```bash
git checkout -b feature/your-feature-name
```

Use descriptive branch names:
- `feature/user-preferences-system`
- `fix/mobile-layout-issue`
- `docs/update-api-documentation`

### 2. Make Your Changes

Follow the existing code patterns and style. See [Code Standards](#code-standards) below.

### 3. Write Tests

All new features must include tests. See [Testing Requirements](#testing-requirements).

### 4. Format Your Code

Before committing, always format your code:

```bash
make fmt
```

### 5. Run Tests

Ensure all tests pass:

```bash
make test
```

For test coverage:

```bash
make cover
```

### 6. Commit Your Changes

Write clear, descriptive commit messages:

```bash
git commit -m "Add user preferences system with theme selection"
```

Good commit message format:
```
Add user preferences system

- Create UserPreference schema and migration
- Add preferences LiveView and template
- Implement theme selection (light/dark)
- Add preference persistence and loading

Closes #123
```

### 7. Push and Create Pull Request

```bash
git push origin feature/your-feature-name
```

Then create a pull request on GitHub with a clear description.

## Code Standards

### Elixir Style Guide

- Follow the [Elixir Style Guide](https://github.com/christopheradams/elixir_style_guide)
- Use `mix format` before committing (enforced by CI)
- Write clear, descriptive function names
- Add `@doc` annotations for public functions
- Use pattern matching effectively

### Phoenix/LiveView Conventions

- Place LiveViews in `lib/valentine_web/live/`
- Place components in `lib/valentine_web/live/[context]/components/`
- Use PrimerLive components where possible for consistency
- Follow existing component patterns

### Database Conventions

- Use descriptive migration names with timestamps
- Always write reversible migrations when possible
- Add indexes for foreign keys and commonly queried fields
- Use proper constraints (NOT NULL, foreign keys, etc.)

### File Organization

```
lib/
├── valentine/
│   ├── composer/           # Domain logic and schemas
│   │   ├── workspace.ex
│   │   ├── user_preference.ex
│   │   └── ...
│   └── prompts/           # AI prompt templates
├── valentine_web/
│   ├── live/              # LiveView modules
│   │   ├── workspace_live/
│   │   ├── user_live/
│   │   └── ...
│   └── components/        # Shared components
priv/
└── repo/
    └── migrations/        # Database migrations
test/
├── valentine/
│   └── composer/          # Context tests
└── valentine_web/
    └── live/              # LiveView tests
```

## Testing Requirements

### Test Coverage

- Aim for 80%+ code coverage
- All new features must include tests
- Test both happy paths and error cases

### Types of Tests

1. **Unit Tests** - Test individual functions and modules
   ```elixir
   # test/valentine/composer/user_preference_test.exs
   defmodule Valentine.Composer.UserPreferenceTest do
     use Valentine.DataCase
     
     alias Valentine.Composer.UserPreference
     
     describe "changeset/2" do
       test "validates required fields" do
         changeset = UserPreference.changeset(%UserPreference{}, %{})
         refute changeset.valid?
       end
     end
   end
   ```

2. **Integration Tests** - Test feature interactions
   ```elixir
   # test/valentine/composer_test.exs
   test "creates user with default preferences" do
     {:ok, user} = Composer.create_user(%{email: "test@example.com"})
     assert user.preferences.theme == "system"
   end
   ```

3. **LiveView Tests** - Test user interactions
   ```elixir
   # test/valentine_web/live/user_live/preferences_test.exs
   test "updates theme preference", %{conn: conn} do
     {:ok, view, _html} = live(conn, "/user/preferences")
     
     view
     |> element("button[phx-click='set-theme'][phx-value-theme='dark']")
     |> render_click()
     
     assert has_element?(view, "[data-theme='dark']")
   end
   ```

### Running Tests

```bash
# Run all tests
make test

# Run specific test file
cd valentine && mix test test/valentine/composer/user_preference_test.exs

# Run with coverage
make cover
```

## Submitting Changes

### Pull Request Checklist

Before submitting a PR, ensure:

- [ ] Code is formatted (`make fmt`)
- [ ] All tests pass (`make test`)
- [ ] New features have tests (80%+ coverage)
- [ ] Documentation is updated
- [ ] Commit messages are clear and descriptive
- [ ] PR description explains the changes
- [ ] Related issue is linked (if applicable)

### PR Description Template

```markdown
## Description
Brief description of the changes

## Related Issue
Closes #123

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
Describe how you tested the changes

## Screenshots (if applicable)
Add screenshots for UI changes

## Checklist
- [ ] Code is formatted
- [ ] Tests pass
- [ ] Documentation updated
- [ ] Ready for review
```

### Code Review Process

1. At least one maintainer must approve
2. All CI checks must pass
3. Resolve all review comments
4. Squash commits if necessary
5. Maintainer will merge when approved

## Implementing Roadmap Features

When implementing features from [NEXT_STEPS.md](NEXT_STEPS.md):

### 1. Choose a Feature

Select a feature from the roadmap that:
- Matches your skill level
- Interests you
- Doesn't have an open PR
- Has clear requirements

### 2. Create an Issue

Before starting work, create an issue:

```markdown
**Feature:** User Preferences System (Phase 1)

**Description:**
Implement user preferences system as described in NEXT_STEPS.md Phase 1.

**Implementation Plan:**
- [ ] Create UserPreference schema
- [ ] Add database migration
- [ ] Create PreferencesLive module
- [ ] Build UI components
- [ ] Write tests
- [ ] Update documentation

**Estimated Time:** 8-12 hours

**Reference:** NEXT_STEPS.md - Phase 1: User Preferences System
```

### 3. Follow the Roadmap Structure

Each feature in NEXT_STEPS.md includes:
- **Files to create/modify** - Follow this structure
- **Tasks** - Use as a checklist
- **Estimated time** - Plan accordingly

### 4. Implementation Guidelines

```elixir
# 1. Start with the schema and migration
defmodule Valentine.Composer.UserPreference do
  use Ecto.Schema
  import Ecto.Changeset

  schema "user_preferences" do
    field :theme, :string, default: "system"
    field :font_size, :string, default: "medium"
    field :language, :string, default: "en"
    
    belongs_to :user, Valentine.Composer.User
    
    timestamps()
  end

  def changeset(preference, attrs) do
    preference
    |> cast(attrs, [:theme, :font_size, :language])
    |> validate_required([:theme, :font_size, :language])
    |> validate_inclusion(:theme, ["light", "dark", "system"])
    |> validate_inclusion(:font_size, ["small", "medium", "large"])
  end
end
```

```elixir
# 2. Add context functions
defmodule Valentine.Composer do
  # ... existing code ...
  
  def get_user_preferences(user_id) do
    # Implementation
  end
  
  def update_user_preferences(user_id, attrs) do
    # Implementation
  end
end
```

```elixir
# 3. Create LiveView
defmodule ValentineWeb.UserLive.Preferences do
  use ValentineWeb, :live_view

  def mount(_params, _session, socket) do
    # Implementation
  end
  
  def handle_event("update-theme", %{"theme" => theme}, socket) do
    # Implementation
  end
end
```

### 5. Write Tests First (TDD)

Consider writing tests before implementation:

```elixir
# Start with failing tests
test "updates user theme preference" do
  user = user_fixture()
  
  {:ok, preferences} = Composer.update_user_preferences(user.id, %{
    theme: "dark"
  })
  
  assert preferences.theme == "dark"
end
```

### 6. Document Your Work

Update relevant documentation:
- Add inline code comments for complex logic
- Update user-facing documentation
- Add examples to function docs
- Update NEXT_STEPS.md to mark completed items

### 7. Manual Testing

Before submitting:
1. Start the server: `make dev`
2. Navigate to your feature
3. Test all user interactions
4. Test on different browsers
5. Test mobile responsiveness
6. Test accessibility (keyboard navigation, screen readers)

## Accessibility Requirements

All UI changes must meet WCAG 2.1 AA standards:

- ✅ Color contrast ratio ≥ 4.5:1 for normal text
- ✅ Touch targets ≥ 44x44 pixels
- ✅ Keyboard navigation support
- ✅ ARIA labels for screen readers
- ✅ Semantic HTML elements
- ✅ Skip links for main content
- ✅ Focus indicators visible

Test with:
- Keyboard-only navigation
- Screen readers (NVDA, JAWS, VoiceOver)
- Browser accessibility tools (axe DevTools)

## Common Patterns

### LiveView Component Pattern

```elixir
defmodule ValentineWeb.UserLive.Components.PreferenceCard do
  use ValentineWeb, :live_component

  def render(assigns) do
    ~H"""
    <div class="preference-card">
      <h3><%= @title %></h3>
      <%= render_slot(@inner_block) %>
    </div>
    """
  end
end
```

### Database Query Pattern

```elixir
def list_workspaces_by_preferences(user_id, theme) do
  from(w in Workspace,
    join: u in assoc(w, :user),
    join: p in assoc(u, :preferences),
    where: u.id == ^user_id and p.theme == ^theme,
    preload: [:user, :preferences]
  )
  |> Repo.all()
end
```

### Event Handling Pattern

```elixir
def handle_event("update-preference", %{"key" => key, "value" => value}, socket) do
  case Composer.update_user_preferences(socket.assigns.current_user.id, %{key => value}) do
    {:ok, preferences} ->
      {:noreply, 
       socket
       |> assign(:preferences, preferences)
       |> put_flash(:info, "Preference updated successfully")}
    
    {:error, %Ecto.Changeset{} = changeset} ->
      {:noreply, assign(socket, :changeset, changeset)}
  end
end
```

## Getting Help

If you need help:

1. **Check Documentation:**
   - [Phoenix LiveView Docs](https://hexdocs.pm/phoenix_live_view)
   - [Elixir Docs](https://hexdocs.pm/elixir)
   - [NEXT_STEPS.md](NEXT_STEPS.md) - Feature roadmap

2. **Search Issues:**
   - Check existing issues for similar problems
   - Look for closed PRs that implemented similar features

3. **Ask Questions:**
   - Create a GitHub Discussion
   - Open an issue with the `question` label
   - Contact: max.neuvians@cds-snc.ca

4. **Community Resources:**
   - [Elixir Forum](https://elixirforum.com)
   - [Phoenix Forum](https://elixirforum.com/c/phoenix-forum)
   - [Stack Overflow](https://stackoverflow.com/questions/tagged/phoenix-framework)

## Code of Conduct

### Our Pledge

We are committed to providing a welcoming and inclusive environment for all contributors.

### Our Standards

- ✅ Be respectful and constructive
- ✅ Welcome newcomers and help them learn
- ✅ Focus on what's best for the project
- ✅ Show empathy towards others
- ❌ No harassment or discriminatory language
- ❌ No trolling or insulting comments
- ❌ No personal or political attacks

### Enforcement

Violations may result in:
1. Warning from maintainers
2. Temporary ban from contributions
3. Permanent ban for repeated violations

Report violations to: max.neuvians@cds-snc.ca

## Recognition

Contributors will be recognized in:
- GitHub contributors page
- Release notes for significant contributions
- Special thanks in documentation

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

## Quick Start Checklist

Ready to contribute? Follow this checklist:

- [ ] Fork the repository
- [ ] Clone your fork locally
- [ ] Run `make setup` to install dependencies
- [ ] Create a feature branch
- [ ] Make your changes
- [ ] Write tests
- [ ] Run `make fmt` to format code
- [ ] Run `make test` to verify tests pass
- [ ] Commit with clear message
- [ ] Push to your fork
- [ ] Create pull request
- [ ] Respond to review feedback

Happy contributing! 🎉

---

*Last updated: October 2025*
