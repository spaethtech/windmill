# Plan: Add New Variable Editor Format Types

## Overview

Add HTML, CSS, JavaScript, TypeScript, Python, and PHP as selectable format types for variable editing in the Windmill frontend. Currently, variables support Plain Text, JSON, and YAML formats. This change is frontend-only and leverages the existing Monaco editor infrastructure.

## Current State

- **File**: `frontend/src/lib/components/VariableEditor.svelte`
- **Current types**: `'plain' | 'json' | 'yaml'`
- **Editor**: Monaco-based `SimpleEditor.svelte` already supports all target languages via:
  - `@codingame/monaco-vscode-standalone-html-language-features`
  - `@codingame/monaco-vscode-standalone-css-language-features`
  - `@codingame/monaco-vscode-standalone-typescript-language-features` (covers JS and TS)
  - `@codingame/monaco-vscode-standalone-languages` (covers Python, PHP, and many others)

## Implementation Steps

### Step 1: Update the `editorKind` Type Definition

**Location**: `VariableEditor.svelte`, line 131

**Current**:
```typescript
let editorKind: 'plain' | 'json' | 'yaml' = $state('plain')
```

**Change to**:
```typescript
let editorKind: 'plain' | 'json' | 'yaml' | 'html' | 'css' | 'javascript' | 'typescript' | 'python' | 'php' = $state('plain')
```

### Step 2: Add Toggle Buttons for New Types

**Location**: `VariableEditor.svelte`, lines 194-200

**Current**:
```svelte
<ToggleButtonGroup bind:selected={editorKind}>
    {#snippet children({ item })}
        <ToggleButton value="plain" label="Plain" {item} />
        <ToggleButton value="json" label="Json" {item} />
        <ToggleButton value="yaml" label="YAML" {item} />
    {/snippet}
</ToggleButtonGroup>
```

**Change to**:
```svelte
<ToggleButtonGroup bind:selected={editorKind}>
    {#snippet children({ item })}
        <ToggleButton value="plain" label="Plain" {item} />
        <ToggleButton value="json" label="Json" {item} />
        <ToggleButton value="yaml" label="YAML" {item} />
        <ToggleButton value="html" label="HTML" {item} />
        <ToggleButton value="css" label="CSS" {item} />
        <ToggleButton value="javascript" label="JS" {item} />
        <ToggleButton value="typescript" label="TS" {item} />
        <ToggleButton value="python" label="Python" {item} />
        <ToggleButton value="php" label="PHP" {item} />
    {/snippet}
</ToggleButtonGroup>
```

### Step 3: Add Editor Blocks for New Types

**Location**: `VariableEditor.svelte`, after line 239 (after the YAML block, before the closing `{/if}`)

Add six new conditional blocks following the existing pattern:

```svelte
{:else if editorKind == 'html'}
    <div class="border rounded mb-4 w-full">
        {#await import('$lib/components/SimpleEditor.svelte')}
            <Loader2 class="animate-spin" />
        {:then Module}
            <Module.default
                bind:this={editor}
                autoHeight
                lang="html"
                bind:code={variable.value}
                fixedOverflowWidgets={false}
                class="bg-surface-tertiary"
            />
        {/await}
    </div>
{:else if editorKind == 'css'}
    <div class="border rounded mb-4 w-full">
        {#await import('$lib/components/SimpleEditor.svelte')}
            <Loader2 class="animate-spin" />
        {:then Module}
            <Module.default
                bind:this={editor}
                autoHeight
                lang="css"
                bind:code={variable.value}
                fixedOverflowWidgets={false}
                class="bg-surface-tertiary"
            />
        {/await}
    </div>
{:else if editorKind == 'javascript'}
    <div class="border rounded mb-4 w-full">
        {#await import('$lib/components/SimpleEditor.svelte')}
            <Loader2 class="animate-spin" />
        {:then Module}
            <Module.default
                bind:this={editor}
                autoHeight
                lang="javascript"
                bind:code={variable.value}
                fixedOverflowWidgets={false}
                class="bg-surface-tertiary"
            />
        {/await}
    </div>
{:else if editorKind == 'typescript'}
    <div class="border rounded mb-4 w-full">
        {#await import('$lib/components/SimpleEditor.svelte')}
            <Loader2 class="animate-spin" />
        {:then Module}
            <Module.default
                bind:this={editor}
                autoHeight
                lang="typescript"
                bind:code={variable.value}
                fixedOverflowWidgets={false}
                class="bg-surface-tertiary"
            />
        {/await}
    </div>
{:else if editorKind == 'python'}
    <div class="border rounded mb-4 w-full">
        {#await import('$lib/components/SimpleEditor.svelte')}
            <Loader2 class="animate-spin" />
        {:then Module}
            <Module.default
                bind:this={editor}
                autoHeight
                lang="python"
                bind:code={variable.value}
                fixedOverflowWidgets={false}
                class="bg-surface-tertiary"
            />
        {/await}
    </div>
{:else if editorKind == 'php'}
    <div class="border rounded mb-4 w-full">
        {#await import('$lib/components/SimpleEditor.svelte')}
            <Loader2 class="animate-spin" />
        {:then Module}
            <Module.default
                bind:this={editor}
                autoHeight
                lang="php"
                bind:code={variable.value}
                fixedOverflowWidgets={false}
                class="bg-surface-tertiary"
            />
        {/await}
    </div>
```

## Files Changed

1. **`frontend/src/lib/components/VariableEditor.svelte`** - Only file that needs modification

## Testing

1. Navigate to the Variables page in Windmill
2. Create a new variable or edit an existing one
3. Verify all nine format buttons are visible: Plain, Json, YAML, HTML, CSS, JS, TS, Python, PHP
4. Test each format by:
   - Switching to the format
   - Entering valid code for that language
   - Verifying syntax highlighting works
   - Verifying the value is saved correctly

## Notes

- **No backend changes required**: The variable value is stored as a plain string regardless of format
- **Format is not persisted**: The format selection is client-side only; when reopening a variable, it defaults to "Plain"
- **Monaco features**: Each language gets full Monaco editor support including syntax highlighting, auto-completion, and error detection
- **Labels**: Using "JS" and "TS" as short labels to keep the toggle button group compact
- **Language identifiers**: Using `python` (not `python3`) and `php` as these are the Monaco language identifiers recognized by the editor
