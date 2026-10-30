<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import tickets from '#lib/data/tickets-4471.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  // Counted the desk's way, as CauseTable does: by cause, with reports the desk read differently
  // footnoted under the cause they are filed under.
  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const issues = tickets.issues as Record<string, number>;
  const tags = (g: { taggedAs: Record<string, number> }) =>
    Object.entries(g.taggedAs)
      .sort((a, b) => b[1] - a[1])
      .map(([tag, n]) => `${tag} ×${n}`)
      .join(' · ');
</script>

{#snippet issueLink(problem: string)}
  {#if issues[problem]}
    <a href="https://github.com/{tickets.repo}/issues/{issues[problem]}">#{issues[problem]}</a>
  {:else}
    <span class="none">none</span>
  {/if}
{/snippet}

<p class="sum">
  <strong>{merged.length}</strong> reports in #{ticket} · <strong>{groups.length + 1}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tagged as</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      {@const moved = readAs(g, notes.oneDayMove)}
      <tr>
        <td><code>{g.cause}</code></td>
        <td>{g.reports.length}</td>
        <td class="muted">{tags(g)}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>{@render issueLink(g.cause)}</td>
      </tr>
      {#if moved}
        <tr class="sub">
          <td><code>one-day-move</code></td>
          <td>{moved}</td>
          <td class="muted" colspan="3">read as a one-day move; counted under <code>{g.cause}</code>, where they are filed</td>
          <td>{@render issueLink('one-day-move')}</td>
        </tr>
      {/if}
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .muted { color: var(--muted); font-size: 0.85em; }
  .sub td:first-child { padding-left: 1.5rem; }
  .none { color: var(--warn); font-weight: 600; }
</style>
