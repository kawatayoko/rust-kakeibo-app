# Copilot Instructions for rust-kakeibo-app

> **Note**: This is a learning project for understanding Rust fundamentals, CLI development, and JSON persistence patterns.

## Project Overview
This is a CLI-based household budget (家計簿) application written in Rust. Users can register income/expense items and aggregate data. All data persists to JSON files using serde serialization.

## Architecture & Module Structure

```
src/
├── main.rs              # Entry point: service type selection (0=register, 1=aggregate)
├── lib.rs               # Module declarations
├── models/mod.rs        # Data models (Item, Category enums)
└── services/
    ├── validate/mod.rs  # Input validation logic
    ├── io/mod.rs        # JSON file I/O operations
    └── register/mod.rs  # User input handlers for registration
```

**Key architectural pattern**: Service-oriented modules with clear separation:
- `models` defines data structures serialized to JSON
- `services::validate` centralizes input validation (uses panic! for invalid input)
- `services::io` handles file operations with graceful fallback (creates new file if missing)
- `services::register` orchestrates user input collection

## Domain Model (models/mod.rs)

**Nested enum pattern for categories**:
```rust
Category::Income(IncomeCategory::Salary)   // 0: 給与, 1: ボーナス, 2: その他
Category::Expense(ExpenseCategory::Food)   // 0: 食費, 1: 趣味, 2: その他
```

**Item structure**: Uses chrono::NaiveDate for date handling (no timezone). Fields are private with a `new()` constructor.

## Input Validation Pattern (services/validate/mod.rs)

**Convention**: All validators panic! on invalid input rather than returning Result. Pattern:
```rust
match input {
    0 | 1 => {},  // Valid values
    _ => panic!("入力値が不正です。")
}
```

Three validators:
- `validate_service_type(0|1)` - Main menu selection
- `validate_register_type(0|1)` - Income(0) or Expense(1)
- `validate_category_type(register_type, 0|1|2)` - Category selection

## File I/O Pattern (services/io/mod.rs)

**Two read strategies**:
1. `read_data_or_create_new_data()` - Returns empty Vec if file missing (for registration)
2. `read_data_or_panic()` - Panics if file missing or empty (for aggregation)

**Write pattern**: `write_to_json()` uses `to_string_pretty()` for human-readable JSON output.

## User Input Pattern (services/register/mod.rs)

**Standard input flow per field**:
1. Print prompt with Japanese instructions
2. Create mutable String buffer
3. `io::stdin().read_line()` with expect() error message
4. Parse with `trim().parse()` for numeric inputs
5. Validate using `services::validate::InputValidator`

**Date input**: Users enter `yyyy-mm-dd` format, parsed via `NaiveDate::from_str()`.

## Known Issues in Codebase

1. **Typo in services/mod.rs**: `pub mod regiter;` should be `register`
2. **Incomplete register/mod.rs**: 
   - `io.stdin()` should be `io::stdin()`
   - Variable `register_type` typo in line 30: `regoster_type`
   - Duplicate `input_price()` function (second should be `input_date()`)
   - Missing variable `mut` in several input functions
3. **Edition warning**: Cargo.toml uses `edition = "2024"` (should be "2021")

## Development Commands

```bash
# Build (checks compilation)
cargo build

# Run the CLI app
cargo run

# Check without building
cargo check
```

No test suite exists yet. To add tests, create functions with `#[cfg(test)]` and `#[test]` attributes.

## Working with This Codebase

- **All user-facing text is in Japanese** - maintain this convention
- **Error handling philosophy**: Use panic! for user input errors, expect() for programmer errors
- **JSON file location**: Not hardcoded in current code - will need to be passed as parameter
- **No async/await**: Synchronous I/O only
- **Dependencies**: Minimal (serde, serde_json, chrono) - keep it lightweight
