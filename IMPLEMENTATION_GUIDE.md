# Phase Implementation Quick Start Guide

This guide provides concrete examples and templates for implementing features from each phase of the roadmap.

## 🎯 Phase 1: Core Functionality

### Example: User Preferences System

#### Step 1: Create the Schema

```elixir
# lib/valentine/composer/user_preference.ex
defmodule Valentine.Composer.UserPreference do
  use Ecto.Schema
  import Ecto.Changeset

  schema "user_preferences" do
    field :theme, :string, default: "system"
    field :font_size, :string, default: "medium"
    field :language, :string, default: "en"
    field :simple_mode, :boolean, default: false
    
    belongs_to :user, Valentine.Composer.User
    
    timestamps()
  end

  @doc false
  def changeset(preference, attrs) do
    preference
    |> cast(attrs, [:theme, :font_size, :language, :simple_mode])
    |> validate_required([:theme, :font_size, :language])
    |> validate_inclusion(:theme, ["light", "dark", "system"])
    |> validate_inclusion(:font_size, ["small", "medium", "large"])
    |> validate_inclusion(:language, ["en", "fr", "es", "de", "zh"])
  end
end
```

#### Step 2: Create the Migration

```elixir
# priv/repo/migrations/20251020000000_create_user_preferences.exs
defmodule Valentine.Repo.Migrations.CreateUserPreferences do
  use Ecto.Migration

  def change do
    create table(:user_preferences) do
      add :theme, :string, null: false, default: "system"
      add :font_size, :string, null: false, default: "medium"
      add :language, :string, null: false, default: "en"
      add :simple_mode, :boolean, null: false, default: false
      add :user_id, references(:users, on_delete: :delete_all), null: false

      timestamps()
    end

    create index(:user_preferences, [:user_id])
    create unique_index(:user_preferences, [:user_id])
  end
end
```

#### Step 3: Add Context Functions

```elixir
# lib/valentine/composer.ex (add these functions)

  alias Valentine.Composer.UserPreference

  def get_user_preferences(user_id) do
    case Repo.get_by(UserPreference, user_id: user_id) do
      nil -> create_default_preferences(user_id)
      preferences -> {:ok, preferences}
    end
  end

  def update_user_preferences(user_id, attrs) do
    case get_user_preferences(user_id) do
      {:ok, preferences} ->
        preferences
        |> UserPreference.changeset(attrs)
        |> Repo.update()
      error -> error
    end
  end

  defp create_default_preferences(user_id) do
    %UserPreference{user_id: user_id}
    |> UserPreference.changeset(%{})
    |> Repo.insert()
  end
```

#### Step 4: Create LiveView

```elixir
# lib/valentine_web/live/user_live/preferences.ex
defmodule ValentineWeb.UserLive.Preferences do
  use ValentineWeb, :live_view
  alias Valentine.Composer

  @impl true
  def mount(_params, _session, socket) do
    user = socket.assigns.current_user
    {:ok, preferences} = Composer.get_user_preferences(user.id)
    
    {:ok,
     socket
     |> assign(:preferences, preferences)
     |> assign(:page_title, "Preferences")}
  end

  @impl true
  def handle_event("update-theme", %{"theme" => theme}, socket) do
    case Composer.update_user_preferences(socket.assigns.current_user.id, %{theme: theme}) do
      {:ok, preferences} ->
        {:noreply,
         socket
         |> assign(:preferences, preferences)
         |> put_flash(:info, "Theme updated successfully")}
      
      {:error, _changeset} ->
        {:noreply, put_flash(socket, :error, "Failed to update theme")}
    end
  end

  @impl true
  def handle_event("update-font-size", %{"size" => size}, socket) do
    case Composer.update_user_preferences(socket.assigns.current_user.id, %{font_size: size}) do
      {:ok, preferences} ->
        {:noreply,
         socket
         |> assign(:preferences, preferences)
         |> put_flash(:info, "Font size updated successfully")}
      
      {:error, _changeset} ->
        {:noreply, put_flash(socket, :error, "Failed to update font size")}
    end
  end

  @impl true
  def handle_event("toggle-simple-mode", _params, socket) do
    new_mode = !socket.assigns.preferences.simple_mode
    
    case Composer.update_user_preferences(socket.assigns.current_user.id, %{simple_mode: new_mode}) do
      {:ok, preferences} ->
        {:noreply,
         socket
         |> assign(:preferences, preferences)
         |> put_flash(:info, "Simple mode #{if new_mode, do: "enabled", else: "disabled"}")}
      
      {:error, _changeset} ->
        {:noreply, put_flash(socket, :error, "Failed to update simple mode")}
    end
  end
end
```

#### Step 5: Create Template

```heex
<!-- lib/valentine_web/live/user_live/preferences.html.heex -->
<div class="container mx-auto px-4 py-8">
  <h1 class="text-3xl font-bold mb-8">User Preferences</h1>

  <div class="space-y-8">
    <!-- Theme Selection -->
    <div class="bg-white rounded-lg shadow p-6">
      <h2 class="text-xl font-semibold mb-4">Theme</h2>
      <div class="flex gap-4">
        <button
          phx-click="update-theme"
          phx-value-theme="light"
          class={"px-4 py-2 rounded #{if @preferences.theme == "light", do: "bg-blue-600 text-white", else: "bg-gray-200"}"}
        >
          Light
        </button>
        <button
          phx-click="update-theme"
          phx-value-theme="dark"
          class={"px-4 py-2 rounded #{if @preferences.theme == "dark", do: "bg-blue-600 text-white", else: "bg-gray-200"}"}
        >
          Dark
        </button>
        <button
          phx-click="update-theme"
          phx-value-theme="system"
          class={"px-4 py-2 rounded #{if @preferences.theme == "system", do: "bg-blue-600 text-white", else: "bg-gray-200"}"}
        >
          System
        </button>
      </div>
    </div>

    <!-- Font Size -->
    <div class="bg-white rounded-lg shadow p-6">
      <h2 class="text-xl font-semibold mb-4">Font Size</h2>
      <div class="flex gap-4">
        <button
          phx-click="update-font-size"
          phx-value-size="small"
          class={"px-4 py-2 rounded text-sm #{if @preferences.font_size == "small", do: "bg-blue-600 text-white", else: "bg-gray-200"}"}
        >
          Small
        </button>
        <button
          phx-click="update-font-size"
          phx-value-size="medium"
          class={"px-4 py-2 rounded text-base #{if @preferences.font_size == "medium", do: "bg-blue-600 text-white", else: "bg-gray-200"}"}
        >
          Medium
        </button>
        <button
          phx-click="update-font-size"
          phx-value-size="large"
          class={"px-4 py-2 rounded text-lg #{if @preferences.font_size == "large", do: "bg-blue-600 text-white", else: "bg-gray-200"}"}
        >
          Large
        </button>
      </div>
    </div>

    <!-- Simple Mode -->
    <div class="bg-white rounded-lg shadow p-6">
      <h2 class="text-xl font-semibold mb-4">Simple Mode</h2>
      <label class="flex items-center gap-3">
        <input
          type="checkbox"
          phx-click="toggle-simple-mode"
          checked={@preferences.simple_mode}
          class="w-5 h-5"
        />
        <span>Enable simplified interface with fewer options</span>
      </label>
    </div>
  </div>
</div>
```

#### Step 6: Add Route

```elixir
# lib/valentine_web/router.ex (add to the authenticated scope)

  scope "/user", ValentineWeb.UserLive do
    pipe_through [:browser, :require_authenticated_user]
    
    live "/preferences", Preferences, :index
  end
```

#### Step 7: Write Tests

```elixir
# test/valentine/composer/user_preference_test.exs
defmodule Valentine.Composer.UserPreferenceTest do
  use Valentine.DataCase
  
  alias Valentine.Composer.UserPreference
  
  describe "changeset/2" do
    test "validates required fields" do
      changeset = UserPreference.changeset(%UserPreference{}, %{})
      assert changeset.valid?
      assert changeset.changes[:theme] == nil
    end
    
    test "validates theme is one of allowed values" do
      changeset = UserPreference.changeset(%UserPreference{}, %{theme: "invalid"})
      refute changeset.valid?
      assert "is invalid" in errors_on(changeset).theme
    end
    
    test "accepts valid theme values" do
      for theme <- ["light", "dark", "system"] do
        changeset = UserPreference.changeset(%UserPreference{}, %{theme: theme})
        assert changeset.valid?
      end
    end
  end
end
```

```elixir
# test/valentine/composer_test.exs (add these tests)
describe "user_preferences" do
  test "get_user_preferences/1 creates default preferences if none exist" do
    user = user_fixture()
    
    assert {:ok, preferences} = Composer.get_user_preferences(user.id)
    assert preferences.theme == "system"
    assert preferences.font_size == "medium"
    assert preferences.language == "en"
    assert preferences.simple_mode == false
  end
  
  test "update_user_preferences/2 updates existing preferences" do
    user = user_fixture()
    {:ok, _} = Composer.get_user_preferences(user.id)
    
    assert {:ok, preferences} = Composer.update_user_preferences(user.id, %{
      theme: "dark",
      font_size: "large"
    })
    
    assert preferences.theme == "dark"
    assert preferences.font_size == "large"
  end
end
```

```elixir
# test/valentine_web/live/user_live/preferences_test.exs
defmodule ValentineWeb.UserLive.PreferencesTest do
  use ValentineWeb.ConnCase
  
  import Phoenix.LiveViewTest
  
  setup do
    user = user_fixture()
    %{user: user, conn: log_in_user(build_conn(), user)}
  end
  
  test "renders preferences page", %{conn: conn} do
    {:ok, _view, html} = live(conn, "/user/preferences")
    assert html =~ "User Preferences"
    assert html =~ "Theme"
    assert html =~ "Font Size"
  end
  
  test "updates theme preference", %{conn: conn} do
    {:ok, view, _html} = live(conn, "/user/preferences")
    
    view
    |> element("button[phx-click='update-theme'][phx-value-theme='dark']")
    |> render_click()
    
    assert render(view) =~ "Theme updated successfully"
  end
  
  test "updates font size preference", %{conn: conn} do
    {:ok, view, _html} = live(conn, "/user/preferences")
    
    view
    |> element("button[phx-click='update-font-size'][phx-value-size='large']")
    |> render_click()
    
    assert render(view) =~ "Font size updated successfully"
  end
  
  test "toggles simple mode", %{conn: conn} do
    {:ok, view, _html} = live(conn, "/user/preferences")
    
    view
    |> element("input[phx-click='toggle-simple-mode']")
    |> render_click()
    
    assert render(view) =~ "Simple mode enabled"
  end
end
```

#### Step 8: Run Migration and Test

```bash
# Run the migration
cd valentine && mix ecto.migrate

# Run tests
mix test test/valentine/composer/user_preference_test.exs
mix test test/valentine/composer_test.exs
mix test test/valentine_web/live/user_live/preferences_test.exs

# Run all tests
mix test

# Check coverage
mix test --cover
```

---

## 📊 Phase 2: Enhanced Visualization

### Example: Score History Chart

#### Step 1: Install ApexCharts (if not already)

```bash
cd assets && npm install apexcharts --save
```

#### Step 2: Create Hook

```javascript
// assets/js/hooks/score_chart_hook.js
import ApexCharts from 'apexcharts';

const ScoreChartHook = {
  mounted() {
    const data = JSON.parse(this.el.dataset.chartData);
    
    const options = {
      series: [{
        name: 'Security Score',
        data: data.scores
      }],
      chart: {
        type: 'line',
        height: 350,
        zoom: {
          enabled: true
        }
      },
      dataLabels: {
        enabled: false
      },
      stroke: {
        curve: 'smooth'
      },
      xaxis: {
        type: 'datetime',
        categories: data.dates
      },
      yaxis: {
        min: 0,
        max: 100
      },
      tooltip: {
        x: {
          format: 'dd MMM yyyy'
        }
      }
    };

    this.chart = new ApexCharts(this.el, options);
    this.chart.render();
  },
  
  updated() {
    const data = JSON.parse(this.el.dataset.chartData);
    this.chart.updateSeries([{
      data: data.scores
    }]);
    this.chart.updateOptions({
      xaxis: {
        categories: data.dates
      }
    });
  },
  
  destroyed() {
    if (this.chart) {
      this.chart.destroy();
    }
  }
};

export default ScoreChartHook;
```

#### Step 3: Register Hook

```javascript
// assets/js/app.js
import ScoreChartHook from './hooks/score_chart_hook';

let liveSocket = new LiveSocket("/live", Socket, {
  params: {_csrf_token: csrfToken},
  hooks: {
    ScoreChart: ScoreChartHook
  }
});
```

#### Step 4: Create Component

```elixir
# lib/valentine_web/live/workspace_live/components/score_history_chart_component.ex
defmodule ValentineWeb.WorkspaceLive.Components.ScoreHistoryChartComponent do
  use ValentineWeb, :live_component
  
  @impl true
  def render(assigns) do
    ~H"""
    <div class="score-history-chart">
      <div class="chart-header mb-4">
        <h3 class="text-xl font-semibold">Security Score History</h3>
        <div class="time-range-selector flex gap-2 mt-2">
          <button
            phx-click="set-time-range"
            phx-value-range="7d"
            phx-target={@myself}
            class={"px-3 py-1 rounded #{if @time_range == "7d", do: "bg-blue-600 text-white", else: "bg-gray-200"}"}
          >
            7 Days
          </button>
          <button
            phx-click="set-time-range"
            phx-value-range="30d"
            phx-target={@myself}
            class={"px-3 py-1 rounded #{if @time_range == "30d", do: "bg-blue-600 text-white", else: "bg-gray-200"}"}
          >
            30 Days
          </button>
          <button
            phx-click="set-time-range"
            phx-value-range="90d"
            phx-target={@myself}
            class={"px-3 py-1 rounded #{if @time_range == "90d", do: "bg-blue-600 text-white", else: "bg-gray-200"}"}
          >
            90 Days
          </button>
          <button
            phx-click="set-time-range"
            phx-value-range="all"
            phx-target={@myself}
            class={"px-3 py-1 rounded #{if @time_range == "all", do: "bg-blue-600 text-white", else: "bg-gray-200"}"}
          >
            All Time
          </button>
        </div>
      </div>
      
      <div
        id={"score-chart-#{@workspace_id}"}
        phx-hook="ScoreChart"
        data-chart-data={Jason.encode!(@chart_data)}
        class="h-96"
      >
      </div>
    </div>
    """
  end
  
  @impl true
  def update(assigns, socket) do
    {:ok,
     socket
     |> assign(assigns)
     |> assign_new(:time_range, fn -> "30d" end)
     |> load_chart_data()}
  end
  
  @impl true
  def handle_event("set-time-range", %{"range" => range}, socket) do
    {:noreply,
     socket
     |> assign(:time_range, range)
     |> load_chart_data()}
  end
  
  defp load_chart_data(socket) do
    # Load score history from database based on time range
    # This is a simplified example
    chart_data = %{
      dates: get_dates_for_range(socket.assigns.time_range),
      scores: get_scores_for_workspace(socket.assigns.workspace_id, socket.assigns.time_range)
    }
    
    assign(socket, :chart_data, chart_data)
  end
  
  defp get_dates_for_range(range) do
    # Implementation to get dates based on range
    []
  end
  
  defp get_scores_for_workspace(workspace_id, range) do
    # Implementation to get scores from database
    []
  end
end
```

---

## 🎓 Phase 3: Educational Content

### Example: FAQ System

See NEXT_STEPS.md for complete implementation details.

Quick implementation checklist:
1. ✅ Create FAQ schema with question, answer, category
2. ✅ Add migration for faqs table
3. ✅ Create FAQ LiveView with search
4. ✅ Add category filtering
5. ✅ Implement "Was this helpful?" feedback
6. ✅ Create admin interface for managing FAQs
7. ✅ Write tests for FAQ functionality

---

## 🤖 Phase 4: AI Enhancement

### Example: AI Security Advisor

See NEXT_STEPS.md for AI integration patterns.

---

## 👥 Phase 5: Social & Collaboration

### Example: User Profiles

Similar pattern to User Preferences - create schema, migration, LiveView, and tests.

---

## 📱 Phase 6: Mobile & Accessibility

### Accessibility Checklist

- [ ] Add ARIA labels to all interactive elements
- [ ] Ensure color contrast meets WCAG AA (4.5:1)
- [ ] Make all features keyboard accessible
- [ ] Test with screen readers (NVDA, JAWS)
- [ ] Add skip links for main content
- [ ] Ensure touch targets are 44x44px minimum
- [ ] Test on mobile devices (iOS, Android)
- [ ] Add semantic HTML throughout
- [ ] Provide text alternatives for images
- [ ] Create accessible forms with proper labels

---

## 💡 Tips for Success

1. **Start Small**: Implement one feature at a time
2. **Test Early**: Write tests before or during development
3. **Follow Patterns**: Use existing code as examples
4. **Document**: Add comments and update docs
5. **Review**: Get code reviewed before merging
6. **Iterate**: Improve based on feedback

---

## 📚 Resources

- [Phoenix LiveView Guide](https://hexdocs.pm/phoenix_live_view)
- [Ecto Documentation](https://hexdocs.pm/ecto)
- [WCAG Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [Elixir Style Guide](https://github.com/christopheradams/elixir_style_guide)

---

*Last updated: October 2025*
