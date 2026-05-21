---
name: dashboard-analytics
description: >
  Utiliser ce skill pour toute création ou modification
  d'un dashboard analytique B2B : KPIs, graphiques
  recharts, tables de données, cartes géographiques,
  filtres de période, navigation entonnoir, conformité.
  TOUJOURS charger pour : dashboard, tableau de bord,
  KPI, graphique, statistiques, carte France, analytics,
  conformité, observations, rapports VGP.
  NE PAS charger pour : formulaires simples, auth,
  pages de contenu, CRUD sans visualisation.
---

# dashboard-analytics — Skill Lovable

Skill de référence pour tous les dashboards analytiques B2B Chopard Equipement.
Combiné avec le skill `ui-ux-pro-max` pour les états de chargement, formulaires et navigation.

---

## 1. DESIGN TOKENS

### Couleurs sémantiques
```css
/* Surfaces */
--surface-page:    #F8F9FB;   /* fond de page */
--surface-card:    #FFFFFF;   /* fond des cards */
--border:          #E2E8F0;   /* bordures */

/* Texte */
--text-primary:    #1A1F2E;
--text-secondary:  #64748B;
--text-muted:      #94A3B8;

/* Brand */
--brand-navy:      #1B2A4A;   /* sidebar, actions primaires */

/* Statuts VGP */
--status-ok:       #10B981;   /* conforme (≥80%) */
--status-warn:     #F59E0B;   /* U2 / partiel (60-79%) */
--status-critical: #EF4444;   /* U1 / non conforme (<60%) */
--status-unknown:  #CBD5E1;   /* non importé */
```

### Palette graphiques recharts (ordre fixe)
```ts
const CHART_COLORS = {
  red:    '#EF4444',
  orange: '#F59E0B',
  green:  '#10B981',
  blue:   '#3B82F6',
  violet: '#8B5CF6',
  gray:   '#94A3B8',
  navy:   '#1B2A4A',
};
```

### Espacement
```
gap cards:    gap-4 (16px)
gap sections: space-y-6 (24px)
padding card: p-5 / p-6
padding page: px-6 py-8 desktop | px-4 py-4 mobile
```

### Border-radius
```
cards:       rounded-xl (12px)
badges:      rounded-md (6px) ou rounded-full
boutons:     rounded-md (6px)
```

### Typographie dashboard
```
Valeur KPI:           text-3xl font-semibold text-[#1A1F2E]
Label KPI:            text-xs font-medium text-[#64748B] uppercase tracking-wide
Titre section card:   text-base font-semibold text-[#1B2A4A]
Sous-titre section:   text-sm text-muted-foreground
Cellule table:        text-sm text-[#1A1F2E]
Badge:                text-xs font-medium
Label axe graphique:  fontSize: 11, fill: '#94A3B8'
```

---

## 2. KPI CARDS

```tsx
interface KpiCardProps {
  label: string;
  value: string | number;
  icon: LucideIcon;
  iconColor: string;
  bgColor: string;           // ex: "bg-blue-50"
  variant?: 'default' | 'critical' | 'warning';
  onClick?: () => void;      // ouvre Sheet drill-down
}

<Card
  className={cn(
    "rounded-xl shadow-sm border transition-shadow",
    variant === 'critical' && "border-red-200 bg-red-50/30",
    variant === 'warning'  && "border-orange-200 bg-orange-50/30",
    onClick && "cursor-pointer hover:shadow-md"
  )}
  onClick={onClick}
>
  <CardContent className="p-5 flex flex-col gap-3">
    <div className={cn("w-10 h-10 rounded-xl flex items-center justify-center", bgColor)}>
      <Icon className="w-5 h-5" style={{ color: iconColor }} aria-hidden="true" />
    </div>
    <div>
      <p className="text-3xl font-semibold text-[#1A1F2E]">{value ?? '—'}</p>
      <p className="text-xs font-medium text-[#64748B] uppercase tracking-wide mt-0.5">{label}</p>
    </div>
  </CardContent>
</Card>
```

**Variants :**
- `default` : fond blanc, icône colorée
- `critical` : `border-red-200 bg-red-50/30` — pour U1, seuils critiquest
- `warning` : `border-orange-200 bg-orange-50/30` — pour U2, avertissements
- **Cliquable** : `cursor-pointer hover:shadow-md` + `onClick` → Sheet drill-down (§6)

---

## 3. GRAPHIQUES RECHARTS

Tous les graphiques :
- Wrappés dans `ChartContainer` de `src/components/ui/chart.tsx`
- `ChartTooltip` avec `ChartTooltipContent` obligatoire
- Titre dans `CardHeader` au-dessus
- 100% français
- État vide si données absentes : div centré text-muted-foreground

### BarChart horizontal (comparaison sites/catégories)
```tsx
const config = { value: { label: "Conformité", color: "#10B981" } } satisfies ChartConfig;

<ChartContainer config={config} className="h-[280px]">
  <BarChart data={data} layout="vertical" margin={{ left: 180, right: 20 }}>
    <CartesianGrid horizontal={false} strokeDasharray="3 3" stroke="#F1F5F9" />
    <YAxis dataKey="name" type="category" width={175} tick={{ fontSize: 12, fill: '#64748B' }} />
    <XAxis type="number" domain={[0, 100]} tickFormatter={v => `${v}%`} tick={{ fontSize: 11, fill: '#94A3B8' }} />
    <Bar dataKey="value" radius={[0, 4, 4, 0]} maxBarSize={20}>
      {data.map((entry, i) => (
        <Cell key={i} fill={entry.value >= 80 ? '#10B981' : entry.value >= 60 ? '#F59E0B' : '#EF4444'} />
      ))}
    </Bar>
    <ChartTooltip content={<ChartTooltipContent formatter={(v) => [`${v}%`, "Conformité"]} />} />
  </BarChart>
</ChartContainer>
```

### BarChart stacked (évolution mensuelle)
```tsx
const config = {
  nouvelles: { label: "Nouvelles",  color: "#EF4444" },
  en_cours:  { label: "En cours",   color: "#F59E0B" },
  cloturees: { label: "Clôturées", color: "#10B981" },
} satisfies ChartConfig;

<ChartContainer config={config} className="h-[280px]">
  <BarChart data={data}>
    <CartesianGrid vertical={false} strokeDasharray="3 3" stroke="#F1F5F9" />
    <XAxis dataKey="month" tick={{ fontSize: 11, fill: '#94A3B8' }} />
    <Bar dataKey="nouvelles" stackId="a" fill="#EF4444" />
    <Bar dataKey="en_cours"  stackId="a" fill="#F59E0B" />
    <Bar dataKey="cloturees" stackId="a" fill="#10B981" radius={[4,4,0,0]} maxBarSize={40} />
    <ChartTooltip content={<ChartTooltipContent />} />
    <ChartLegend content={<ChartLegendContent />} />
  </BarChart>
</ChartContainer>
```

### PieChart / Donut
```tsx
const config = {
  U1: { label: "Défaut urgent (U1)", color: "#EF4444" },
  U2: { label: "Non-conforme (U2)",  color: "#F59E0B" },
  U3: { label: "Observation (U3)",   color: "#FBBF24" },
} satisfies ChartConfig;

const total = data.reduce((s, d) => s + d.value, 0);

<ChartContainer config={config} className="h-[280px]">
  <PieChart>
    <Pie data={data.filter(d => d.value > 0)} dataKey="value" nameKey="name"
         innerRadius={60} outerRadius={100} paddingAngle={3}>
      <Label content={({ viewBox }) => {
        const { cx, cy } = viewBox as { cx: number; cy: number };
        return (
          <text x={cx} y={cy} textAnchor="middle" dominantBaseline="middle">
            <tspan x={cx} y={cy - 6} fontSize={22} fontWeight={600} fill="#1A1F2E">{total}</tspan>
            <tspan x={cx} y={cy + 14} fontSize={11} fill="#94A3B8">observations</tspan>
          </text>
        );
      }} />
    </Pie>
    <ChartTooltip content={<ChartTooltipContent />} />
    <ChartLegend content={<ChartLegendContent />} />
  </PieChart>
</ChartContainer>
```

### Règles graphiques
- Jamais sans `CardHeader` avec titre
- Jamais sans `ChartTooltip`
- Toutes les couleurs dans `chartConfig`, jamais inline dans les composants
- État vide : `<div className="h-[280px] flex items-center justify-center text-muted-foreground text-sm">Aucune donnée disponible</div>`
- `ChartLegend` toujours en bas

---

## 4. TABLES shadcn

```tsx
// Pattern standard : Table dans Card, max 8 lignes + bouton voir plus
<Card className="rounded-xl shadow-sm">
  <CardHeader className="pb-2">
    <CardTitle className="text-base font-semibold text-[#1B2A4A]">{title}</CardTitle>
    {subtitle && <CardDescription>{subtitle}</CardDescription>}
  </CardHeader>
  <CardContent className="p-0">
    <Table>
      <TableHeader>
        <TableRow className="bg-[#F8F9FB] hover:bg-[#F8F9FB]">
          <TableHead className="text-xs font-medium text-[#94A3B8] uppercase tracking-wide">Col</TableHead>
        </TableRow>
      </TableHeader>
      <TableBody>
        {rows.slice(0, 8).map(row => (
          <TableRow key={row.id} className="cursor-pointer hover:bg-[#F8F9FB] transition-colors"
                    onClick={() => handleRowClick(row)}>
            {/* cells — valeur null/undefined → afficher '—' */}
          </TableRow>
        ))}
      </TableBody>
    </Table>
    {rows.length > 8 && (
      <div className="px-4 py-3 border-t border-[#E2E8F0]">
        <Button variant="ghost" size="sm" className="text-[#64748B] w-full">
          Voir les {rows.length - 8} autres
        </Button>
      </div>
    )}
  </CardContent>
</Card>
```

### Badges statuts
```tsx
// U1/U2/U3 : toujours variant="outline" + classes colorées
<Badge variant="outline" className="bg-red-100 text-red-700 border-red-200">U1</Badge>
<Badge variant="outline" className="bg-orange-100 text-orange-700 border-orange-200">U2</Badge>
<Badge variant="outline" className="bg-yellow-100 text-yellow-700 border-yellow-200">U3</Badge>
<Badge variant="outline" className="bg-green-100 text-green-700 border-green-200">Conforme</Badge>
```

### Progress bar conformité (mini inline)
```tsx
<div className="flex items-center gap-2">
  <div className="flex-1 h-1.5 bg-[#E2E8F0] rounded-full overflow-hidden">
    <div className="h-full rounded-full transition-all" style={{
      width: `${score}%`,
      backgroundColor: score >= 80 ? '#10B981' : score >= 60 ? '#F59E0B' : '#EF4444'
    }} />
  </div>
  <span className="text-xs font-medium w-8 text-right">{score ?? '—'}%</span>
</div>
```

---

## 5. FILTRE PÉRIODE

```tsx
// Pills horizontales — top-right du header, max 220px total
const PERIODS = [
  { key: 'month', label: 'Ce mois' },
  { key: '6m',    label: '6 mois' },
  { key: '2025',  label: '2025' },
  { key: '2024',  label: '2024' },
];

<div className="flex gap-1">
  {PERIODS.map(p => (
    <button key={p.key} onClick={() => setPeriod(p.key)}
      className={cn(
        "px-3 py-1 rounded-full text-xs font-medium transition-colors",
        period === p.key ? "bg-[#1B2A4A] text-white" : "bg-[#F1F5F9] text-[#64748B] hover:bg-[#E2E8F0]"
      )}>{p.label}</button>
  ))}
</div>
```

**Règle** : affecte uniquement graphiques et tables. Les KPI cards affichent toujours l'état actuel.

---

## 6. NAVIGATION ENTONNOIR

### KPI → Sheet drill-down
```tsx
<Sheet open={sheetOpen} onOpenChange={setSheetOpen}>
  <SheetContent side="right" className="w-[480px] sm:max-w-[480px] overflow-y-auto">
    <SheetHeader className="pb-4 border-b border-[#E2E8F0]">
      <SheetTitle className="text-[#1B2A4A]">{kpiLabel} — Détail</SheetTitle>
    </SheetHeader>
    <div className="mt-6 space-y-6">
      {/* Sous-métriques : BarChart horizontal par Mode A/B, par catégorie… */}
    </div>
  </SheetContent>
</Sheet>
// KPI card onClick → setSheetOpen(true)
```

### Ligne de table → navigate
```tsx
import { useNavigate } from '@tanstack/react-router';
const navigate = useNavigate();

<TableRow className="cursor-pointer hover:bg-[#F8F9FB]"
  onClick={() => navigate({ to: '/sites/$siteId', params: { siteId: row.id } })}>
// JAMAIS modal pour les lignes de table — toujours navigate
```

### Breadcrumb
```tsx
// Visible dès niveau 2 (§4 du skill ui-ux-pro-max)
<Breadcrumb>
  <BreadcrumbList>
    <BreadcrumbItem><BreadcrumbLink asChild><Link to="/">Tableau de bord</Link></BreadcrumbLink></BreadcrumbItem>
    <BreadcrumbSeparator />
    <BreadcrumbItem><BreadcrumbPage>{siteName}</BreadcrumbPage></BreadcrumbItem>
  </BreadcrumbList>
</Breadcrumb>
```

---

## 7. CARTE GÉOGRAPHIQUE (France)

### Librairie : react-simple-maps
```tsx
import { ComposableMap, Geographies, Geography, Marker, ZoomableGroup } from 'react-simple-maps';
// Vérifier package.json avant — installer si absent

const GEO_URL = 'https://raw.githubusercontent.com/gregoiredavid/france-geojson/master/regions-version-simplifiee.geojson';
```

### Couleurs bulles (VGP)
```ts
const getBubbleColor = (site: SiteGeo) => {
  if (!site.imported || site.score === null) return '#CBD5E1'; // gris
  if (site.score >= 80) return '#10B981'; // vert
  if (site.score >= 60) return '#F59E0B'; // orange
  return '#EF4444';                       // rouge
};

const getBubbleRadius = (count: number) =>
  Math.min(16, Math.max(4, Math.sqrt(count || 1) * 1.5));
```

### JSX carte complète
```tsx
<Card className="rounded-xl shadow-sm">
  <CardHeader className="pb-2">
    <CardTitle className="text-base font-semibold text-[#1B2A4A]">Conformité par site</CardTitle>
    <CardDescription>85 positions · cliquer pour le détail</CardDescription>
  </CardHeader>
  <CardContent className="p-0">
    <ComposableMap projection="geoMercator"
      projectionConfig={{ center: [2.5, 46.5], scale: 2600 }}
      className="h-[340px] w-full">
      <ZoomableGroup center={[2.5, 46.5]} zoom={1}>
        <Geographies geography={GEO_URL}>
          {({ geographies }) => geographies.map(geo => (
            <Geography key={geo.rsmKey} geography={geo}
              fill="#F1F5F9" stroke="#E2E8F0" strokeWidth={0.5}
              style={{ default:{outline:'none'}, hover:{outline:'none'}, pressed:{outline:'none'} }} />
          ))}
        </Geographies>
        {sites.map(site => (
          <Marker key={site.id} coordinates={[site.lng, site.lat]}>
            <TooltipProvider>
              <Tooltip delayDuration={0}>
                <TooltipTrigger asChild>
                  <circle r={getBubbleRadius(site.equipment_count)}
                    fill={getBubbleColor(site)} fillOpacity={0.85}
                    stroke="white" strokeWidth={1}
                    style={{ cursor: site.imported ? 'pointer' : 'default' }}
                    onClick={() => site.imported && navigate({ to: '/sites/$siteId', params: { siteId: site.id.toString() } })} />
                </TooltipTrigger>
                <TooltipContent>
                  <p className="font-medium">{site.name}</p>
                  <p className="text-xs text-muted-foreground">
                    {site.imported && site.score !== null
                      ? `Conformité : ${site.score}% · U1: ${site.u1} · U2: ${site.u2}`
                      : 'Pas encore importé'}
                  </p>
                </TooltipContent>
              </Tooltip>
            </TooltipProvider>
          </Marker>
        ))}
      </ZoomableGroup>
    </ComposableMap>
  </CardContent>
</Card>
```

### Fallback si react-simple-maps absent
SVG inline `viewBox="0 0 600 700"` avec contour France simplifié.
Projection linéaire : `x = (lng - (-5)) / (9.5 - (-5)) * 500 + 50`, `y = (51 - lat) / (51 - 41.5) * 600 + 50`

---

## 8. MOCK DATA PATTERN

```tsx
// En haut du composant, avant le JSX
import sitesGeo from "@/../../data/chopard_sites_geo.json";

const MOCK_DATA = {
  // Utiliser les vrais sites du JSON pour la carte
  // Générer des données mensuelles réalistes (12 mois)
  monthlyObs: Array.from({ length: 12 }, (_, i) => ({
    month: new Date(2025, i, 1).toLocaleDateString('fr-FR', { month: 'short', year: '2-digit' }),
    nouvelles: Math.floor(Math.random() * 15) + 5,
    en_cours:  Math.floor(Math.random() * 25) + 10,
    cloturees: Math.floor(Math.random() * 20) + 8,
  })),
  // Distribution catégories réaliste
  categoryDist: [
    { name: 'Levage',       value: 38, fill: '#3B82F6' },
    { name: 'Incendie',     value: 29, fill: '#EF4444' },
    { name: 'Portes',       value: 19, fill: '#10B981' },
    { name: 'Électricité', value: 14, fill: '#F59E0B' },
    { name: 'Ventilation',  value: 8,  fill: '#8B5CF6' },
    { name: 'Autre',        value: 6,  fill: '#94A3B8' },
  ],
};

// Sélection données
const { sites: realSites = [] } = useDashboard() ?? {};
const isMock = realSites.length < 3;
// La carte utilise toujours sitesGeo (toutes les 85 positions)

// Banner mock
{isMock && (
  <Alert className="mt-4 border-[#E2E8F0] bg-[#F8F9FB]">
    <AlertDescription className="text-xs text-[#64748B]">
      Mode démo — certaines données sont simulées.
      Importez des rapports VGP pour voir vos données réelles.
    </AlertDescription>
  </Alert>
)}
```

---

## 9. RÈGLES ABSOLUES

| Règle | ❌ Interdit | ✅ Obligatoire |
|---|---|---|
| Composants UI | Tout autre library | `shadcn/ui` exclusivement |
| Icônes | Heroicons, FA, Tabler | `lucide-react` exclusivement |
| Graphiques | D3, Highcharts | `recharts` via `ChartContainer` |
| Langue UI | Anglais | 100% français |
| Routing | react-router-dom | `@tanstack/react-router` |
| Dark mode | — | Ne pas implémenter |
| Fond de page | Blanc pur | `bg-[#F8F9FB]` |
| Valeur null/undefined | Afficher rien | Afficher `—` |
| Toasts | react-hot-toast | `sonner` |

### Responsive
```
KPI grid:      grid-cols-2 sm:grid-cols-4
Graphiques:    grid-cols-1 lg:grid-cols-3
Carte + top5:  grid-cols-1 lg:grid-cols-2
Tables:        overflow-x-auto sur mobile
```

### Accessibilité
- Icônes interactives seules : `aria-label` obligatoire
- Icônes décoratives : `aria-hidden="true"`
- Contrastes : 4.5:1 minimum sur tous les textes

### États des graphiques et tables
- Chargement → `Skeleton` (jamais spinner plein écran) — voir skill `ui-ux-pro-max` §3
- Vide → div centré avec icône + message + CTA — voir skill `ui-ux-pro-max` §3
- Erreur → `Alert variant="destructive"` + retry — voir skill `ui-ux-pro-max` §3

### Performance
```tsx
// Calculs d'agrégation : useMemo obligatoire
const kpis = useMemo(() => {
  const imported = sitesGeo.filter(s => s.imported);
  const totalScore = imported.reduce((sum, s) => sum + (s.score ?? 0), 0);
  return {
    conformite: imported.length > 0 ? Math.round(totalScore / imported.length) : 0,
    u1Total: sitesGeo.reduce((sum, s) => sum + s.u1, 0),
    u2Total: sitesGeo.reduce((sum, s) => sum + s.u2, 0),
    importedCount: imported.length,
  };
}, [sitesGeo]);
```
