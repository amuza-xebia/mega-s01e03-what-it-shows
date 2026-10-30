<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // The desk's own view (causes.ts), with the ticket filed for each problem.
  // `issues`: cause → ticket; `oneDayMove`: the ticket for the reports the desk reads as a one-day move.
  let {
    ticket,
    issues = {},
    oneDayMove,
  }: { ticket: number; issues?: Record<string, { url: string; state: string }>; oneDayMove?: { url: string; state: string } } =
    $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const counts = (n: Record<string, number>) =>
    Object.entries(n)
      .sort((a, b) => b[1] - a[1])
      .map(([k, v]) => `${k} ×${v}`)
      .join(' · ');
  const label = (u: string) => '#' + u.split('/').pop();
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{groups.length}</strong>
  {groups.length === 1 ? 'cause' : 'causes'}
</p>

<table>
  <thead>
    <tr><th>cause</th><th>reports</th><th>tagged as</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      {@const t = issues[g.cause]}
      <tr class:open={!t}>
        <td><code>{g.cause}</code></td>
        <td>{g.reports.length}</td>
        <td class="small">{counts(g.taggedAs)}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>{#if t}<a href={t.url}>{label(t.url)}</a> · {t.state}{:else}no ticket yet{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

{#each groups as g (g.cause)}
  {@const moved = readAs(g, notes.oneDayMove)}
  {#if moved}
    <p class="small">
      {moved} of the <code>{g.cause}</code> reports read as a one-day move, not a swap; counted under
      <code>{g.cause}</code> by tenant, and filed on their own as
      {#if oneDayMove}<a href={oneDayMove.url}>{label(oneDayMove.url)}</a> · {oneDayMove.state}{:else}no ticket yet{/if}.
    </p>
  {/if}
{/each}

<style>
  .sum { font-size: 1.1rem; }
  .small { color: var(--muted); font-size: 0.85em; }
  .open td { background: var(--warn-bg); }
  .open td:last-child { color: var(--warn); font-weight: 600; }
</style>
