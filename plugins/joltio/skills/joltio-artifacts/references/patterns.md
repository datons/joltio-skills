# Artifact patterns

Choose the smallest interaction model that expresses the decision the user needs to make.

## Governed function import

```svelte
<script lang="ts">
  import runSpreads from 'joltio:function/esios_market_spread_summary';
  let result = $state<{ rows: Record<string, unknown>[] }>({ rows: [] });
  $effect(() => { runSpreads().then((value) => result = value); });
</script>
```

The import is static so Joltio can derive dependencies, validate authorization, and audit execution before publishing.

## Selection and callbacks

Keep selections in component state and derive every linked view from that state. Use ordinary Svelte event handlers for filters, hover details, drill-down, tab changes, and scenario toggles. A callback may transform already-authorized rows locally; it must not fetch new data except through a governed function import.

## Heatmap

Use a heatmap for two categorical or temporal dimensions where color carries one numeric measure. Include a visible legend, explicit units, keyboard-readable cell labels, and a table fallback when exact values matter.

## Market report

For market spreads, show the coverage window and sources first, then the SPOT reference, intraday series, spread distribution, extrema, and a compact table. Run the ESIOS function before authoring so labels and units match its actual output schema.

## Managed layout

Declare movable regions with stable block identifiers and keep their positions in `src/layout.json`. Do not infer layout from pixel coordinates embedded across components. Freeform narrative and bespoke interactions remain in Svelte source.
