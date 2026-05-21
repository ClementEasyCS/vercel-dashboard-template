---
name: dashboard-analytics
description: >
  Utiliser ce skill pour toute création ou modification
  d'un dashboard analytique B2B : KPIs, graphiques
  recharts, tables de données, cartes géographiques,
  filtres de période, navigation entonnoir, conformité.
  TOUJOURS charger pour : dashboard, tableau de bord,
  KPI, graphique, statistiques, carte France, analytics,
  observations, rapports VGP.
  NE PAS charger pour : formulaires simples, auth,
  pages de contenu, CRUD sans visualisation.
---

# dashboard-analytics — Skill Lovable

Skill de référence pour tous les dashboards analytiques B2B du projet Chopard Equipement.
Extraire les patterns de `vercel-dashboard-template` + conventions `ui-ux-pro-max`.

---

## 1. DESIGN TOKENS

### Couleurs sémantiques (CSS variables)
```css
/* Fond et surfaces */
--surface-page:    #F8F9FB;   /* fond de page (bg-[#F8F9FB]) */
--surface-card:    #FFFFFF;   /* fond des cards */
--border:          #E2E8F0;   /* bordures (border-[#E2E8F0] ou border-border) */

/* Texte */
--text-primary:    #1A1F2E;   /* titres, valeurs */
--text-secondary:  #64748B;   /* labels, sous-titres */
--text-muted:      #94A3B8;   /* placeholders, texte désactivé */

/* Brand */
--brand-navy:      #1B2A4A;   /* sidebar, primary actions */
--brand-navy-light:#2D4A7A;

/* Statuts */
--status-ok:       #10B981;   /* conforme, vert */
--status-warn:     #F59E0B;   /* U2, orange */
--status-critical: #EF4444;   /* U1, rouge */
--status-unknown:  #94A3B8;   /* non importé, gris */
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

### Espacements
```
gap cards:    gap-4 (16px)
gap sections: space-y-6 (24px)
padding card: p-5 ou p-6
padding page: px-6 py-8 (desktop), px-4 py-4 (mobile)
```

### Border-radius
```
cards:    rounded-xl (12px)
badges:   rounded-md (6px) ou rounded-full
boutons:  rounded-md (6px)
```

### Ombres
```
card shadow:       shadow-sm (0 1px 2px rgba(0,0,0,0.05))
card hover shadow: shadow-md
```

### Typographie
```
Valeur KPI principale:  text-3xl font-semibold text-[#1A1F2E]
Titre KPI:              text-sm font-medium text-[#64748B] uppercase tracking-wide
Titre section card:     text-base font-semibold text-[#1B2A4A]
Sous-titre section:     text-sm text-muted-foreground
Cellule table:          text-sm text-[#1A1F2E]
Badge:                  text-xs font-medium
Label axe graphique:    fontSize: 11, fill: '#94A3B8'
```

---

## 2. KPI CARDS

Toujours utiliser `Card` shadcn. Layout flex-col.

```tsx
interface KpiCardProps {
  label: string;
  value: string | number;
  icon: LucideIcon;
  iconColor: string;
  bgColor: string;           // ex: "bg-blue-50"
  delta?: { value: string; positive: boolean };  // variation vs période
  variant?: 'default' | 'critical' | 'warning';  // critical = bordure rouge, warning = bordure orange
  onClick?: () => void;      // ouvre un Sheet drill-down
}

// Structure JSX
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
    {/* Icône */}
    <div className={cn("w-10 h-10 rounded-xl flex items-center justify-center", bgColor)}>
      <Icon className="w-5 h-5" style={{ color: iconColor }} />
    </div>
    {/* Valeur */}
    <div>
      <p className="text-3xl font-semibold text-[#1A1F2E]">{value}</p>
      <p className="text-xs font-medium text-[#64748B] uppercase tracking-wide mt-0.5">{label}</p>
    </div>
    {/* Delta optionnel */}
    {delta && (
      <span className={cn("text-xs font-medium",
        delta.positive ? "text-green-600" : "text-red-500"
      )}>
        {delta.positive ? '↑' : '↓'} {delta.value}
      </span>
    )}
  </CardContent>
</Card>
```

### Variants KPI
- **default** : fond blanc, icône colorée, pas de bordure spéciale
- **critical** : fond rouge très léger (`bg-red-50/30`), bordure `border-red-200` — pour U1, alertes
- **warning** : fond orange très léger (`bg-orange-50/30`), bordure `border-orange-200` — pour U2
- **cliquable** : `cursor-pointer hover:shadow-md` + `onClick` → ouvre Sheet drill-down (§6)

---

## 3. GRAPHIQUES RECHARTS

Tous les graphiques :
- Wrappés dans `ChartContainer` de `src/components/ui/chart.tsx`
- `ResponsiveContainer` implicite via ChartContainer
- `ChartTooltip` avec `ChartTooltipContent` obligatoire
- Titre + description dans `CardHeader` au-dessus
- 100% français dans les labels

### BarChart (vertical — comparaison catégories)
```tsx
const config = {
  value: { label: "Conformité", color: "#10B981" },
} satisfies ChartConfig;

<ChartContainer config={config} className="h-[280px]">
  <BarChart data={data} layout="vertical" margin={{ left: 180, right: 20 }}>
    <CartesianGrid horizontal={false} strokeDasharray="3 3" stroke="#F1F5F9" />
    <YAxis dataKey="name" type="category" width={175} tick={{ fontSize: 12, fill: '#64748B' }} />
    <XAxis type="number" domain={[0, 100]} tickFormatter={v => `${v}%`} tick={{ fontSize: 11, fill: '#94A3B8' }} />
    <Bar dataKey="value" radius={[0, 4, 4, 0]} maxBarSize={20}>
      {data.map((entry, i) => (
        <Cell key={i} fill={
          entry.value >= 80 ? '#10B981' :
          entry.value >= 60 ? '#F59E0B' : '#EF4444'
        } />
      ))}
    </Bar>
    <ChartTooltip content={<ChartTooltipContent formatter={(v) => [`${v}%`, "Conformité"]} />} />
  </BarChart>
</ChartContainer>
```

### BarChart stacked (évolution mensuelle)
```tsx
const config = {
  nouvelles:  { label: "Nouvelles",  color: "#EF4444" },
  en_cours:   { label: "En cours",   color: "#F59E0B" },
  cloturees:  { label: "Clôturées",  color: "#10B981" },
} satisfies ChartConfig;

<ChartContainer config={config} className="h-[280px]">
  <BarChart data={data}>
    <CartesianGrid vertical={false} strokeDasharray="3 3" stroke="#F1F5F9" />
    <XAxis dataKey="month" tick={{ fontSize: 11, fill: '#94A3B8' }} />
    <Bar dataKey="nouvelles" stackId="a" fill="#EF4444" radius={[0,0,0,0]} />
    <Bar dataKey="en_cours"  stackId="a" fill="#F59E0B" radius={[0,0,0,0]} />
    <Bar dataKey="cloturees" stackId="a" fill="#10B981" radius={[4,4,0,0]} maxBarSize={40} />
    <ChartTooltip content={<ChartTooltipContent />} />
    <ChartLegend content={<ChartLegendContent />} />
  </BarChart>
</ChartContainer>
```

### PieChart / Donut
```tsx
const config = {
  U1: { label: "Défaut urgent (U1)",  color: "#EF4444" },
  U2: { label: "Non-conforme (U2)",   color: "#F59E0B" },
  U3: { label: "Observation (U3)",    color: "#FBBF24" },
} satisfies ChartConfig;

const total = data.reduce((s, d) => s + d.value, 0);

<ChartContainer config={config} className="h-[280px]">
  <PieChart>
    <Pie
      data={data.filter(d => d.value > 0)}
      dataKey="value"
      nameKey="name"
      innerRadius={60}
      outerRadius={100}
      paddingAngle={3}
    >
      {/* Label centré dans le donut */}
      <Label
        content={({ viewBox }) => {
          const { cx, cy } = viewBox as { cx: number; cy: number };
          return (
            <text x={cx} y={cy} textAnchor="middle" dominantBaseline="middle">
              <tspan x={cx} y={cy - 6} fontSize={22} fontWeight={600} fill="#1A1F2E">{total}</tspan>
              <tspan x={cx} y={cy + 14} fontSize={11} fill="#94A3B8">observations</tspan>
            </text>
          );
        }}
      />
    </Pie>
    <ChartTooltip content={<ChartTooltipContent />} />
    <ChartLegend content={<ChartLegendContent />} />
  </PieChart>
</ChartContainer>
```

### LineChart
```tsx
// Max 2 courbes. Points visibles. Grille horizontale uniquement.
<LineChart data={data}>
  <CartesianGrid vertical={false} strokeDasharray="3 3" stroke="#F1F5F9" />
  <XAxis dataKey="month" tick={{ fontSize: 11, fill: '#94A3B8' }} />
  <YAxis tick={{ fontSize: 11, fill: '#94A3B8' }} />
  <Line type="monotone" dataKey="value" stroke="#1B2A4A" strokeWidth={2} dot={{ r: 3 }} />
  <ChartTooltip content={<ChartTooltipContent />} />
</LineChart>
```

### Règles absolues graphiques
- Jamais de graphique sans `CardHeader` avec titre
- Jamais de graphique sans `ChartTooltip`
- État vide : `<div className="h-[280px] flex items-center justify-center text-muted-foreground text-sm">Aucune donnée disponible</div>`
- Toutes les couleurs dans `chartConfig`, jamais inline
- `ChartLegendContent` toujours en bas (`verticalAlign="bottom"`)

---

## 4. TABLES shadcn

```tsx
// Pattern standard : Table dans Card, max 8 lignes visibles
<Card className="rounded-xl shadow-sm">
  <CardHeader className="pb-2">
    <CardTitle className="text-base font-semibold text-[#1B2A4A]">{title}</CardTitle>
    {subtitle && <CardDescription>{subtitle}</CardDescription>}
  </CardHeader>
  <CardContent className="p-0">
    <Table>
      <TableHeader>
        <TableRow className="bg-[#F8F9FB] hover:bg-[#F8F9FB]">
          {columns.map(col => (
            <TableHead key={col.key} className="text-xs font-medium text-[#94A3B8] uppercase tracking-wide">
              {col.sortable ? (
                <Button variant="ghost" size="sm" className="-ml-3 h-8" onClick={() => handleSort(col.key)}>
                  {col.label}
                  {sortKey === col.key ? (sortDir === 'asc' ? <ArrowUp className="ml-1 h-3 w-3" /> : <ArrowDown className="ml-1 h-3 w-3" />) : <ArrowUpDown className="ml-1 h-3 w-3" />}
                </Button>
              ) : col.label}
            </TableHead>
          ))}
        </TableRow>
      </TableHeader>
      <TableBody>
        {rows.slice(0, 8).map(row => (
          <TableRow
            key={row.id}
            className="cursor-pointer hover:bg-[#F8F9FB] transition-colors"
            onClick={() => onRowClick(row)}
          >
            {/* cells */}
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

### Badges statuts dans tables
```tsx
const StatusBadge = ({ level }: { level: 'U1' | 'U2' | 'U3' | 'ok' }) => {
  const styles = {
    U1: 'bg-red-100 text-red-700 border-red-200',
    U2: 'bg-orange-100 text-orange-700 border-orange-200',
    U3: 'bg-yellow-100 text-yellow-700 border-yellow-200',
    ok: 'bg-green-100 text-green-700 border-green-200',
  };
  return <Badge variant="outline" className={styles[level]}>{level}</Badge>;
};
```

### Progress bar conformité (mini)
```tsx
<div className="flex items-center gap-2">
  <div className="flex-1 h-1.5 bg-[#E2E8F0] rounded-full overflow-hidden">
    <div
      className="h-full rounded-full transition-all"
      style={{
        width: `${score}%`,
        backgroundColor: score >= 80 ? '#10B981' : score >= 60 ? '#F59E0B' : '#EF4444'
      }}
    />
  </div>
  <span className="text-xs font-medium w-8 text-right">{score}%</span>
</div>
```

---

## 5. FILTRE PÉRIODE

Positionné `top-right` du `CardHeader` ou du header de page.
Jamais full-width. Max 220px total.

```tsx
// Pills horizontales — pills actives = bg-[#1B2A4A] text-white
const PERIODS = [
  { key: 'month', label: 'Ce mois' },
  { key: '6m',    label: '6 mois' },
  { key: '2025',  label: '2025' },
  { key: '2024',  label: '2024' },
];

<div className="flex gap-1">
  {PERIODS.map(p => (
    <button
      key={p.key}
      onClick={() => setPeriod(p.key)}
      className={cn(
        "px-3 py-1 rounded-full text-xs font-medium transition-colors",
        period === p.key
          ? "bg-[#1B2A4A] text-white"
          : "bg-[#F1F5F9] text-[#64748B] hover:bg-[#E2E8F0]"
      )}
    >
      {p.label}
    </button>
  ))}
</div>
```

**Règle** : le filtre période affecte uniquement les graphiques et les tables.
Les KPI cards affichent toujours l'état actuel (non filtré).

---

## 6. NAVIGATION ENTONNOIR (drill-down)

### KPI → Sheet
```tsx
const [sheetOpen, setSheetOpen] = useState(false);

// KPI card avec onClick → ouvre Sheet
<Sheet open={sheetOpen} onOpenChange={setSheetOpen}>
  <SheetContent side="right" className="w-[480px] sm:max-w-[480px]">
    <SheetHeader>
      <SheetTitle className="text-[#1B2A4A]">{kpiLabel} — Détail</SheetTitle>
      <SheetDescription>{kpiSubtitle}</SheetDescription>
    </SheetHeader>
    <div className="mt-6 space-y-6">
      {/* Sous-métriques avec mini barres */}
      {/* Ex: Mode A / Mode B séparément */}
      {/* Ex: Catégories (Levage, Incendie, Portes…) */}
    </div>
  </SheetContent>
</Sheet>
```

### Ligne de table → navigate
```tsx
// Toujours navigate, jamais modal, pour les lignes de table
import { useNavigate } from '@tanstack/react-router';
const navigate = useNavigate();

<TableRow
  className="cursor-pointer hover:bg-[#F8F9FB]"
  onClick={() => navigate({ to: '/sites/$siteId', params: { siteId: row.id } })}
>
```

### Breadcrumb
```tsx
// Visible dès que ≥ 2 niveaux de navigation
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

### Librairie : react-simple-maps (package à installer si absent)
```tsx
import { ComposableMap, Geographies, Geography, Marker, ZoomableGroup } from 'react-simple-maps';

// GeoJSON France métropolitaine
const GEO_URL = 'https://raw.githubusercontent.com/gregoiredavid/france-geojson/master/regions-version-simplifiee.geojson';

// Projection centrée sur la France
const projection = {
  center: [2.5, 46.5],   // centre France
  scale: 2600,
};
```

### Couleurs des bulles
```ts
const getBubbleColor = (site: SiteGeo) => {
  if (!site.imported) return '#CBD5E1';        // gris — non importé
  if (site.score === null) return '#CBD5E1';
  if (site.score >= 80)    return '#10B981';   // vert
  if (site.score >= 60)    return '#F59E0B';   // orange
  return '#EF4444';                            // rouge
};

// Taille bulle proportionnelle aux équipements
const getBubbleRadius = (equipmentCount: number) =>
  Math.min(16, Math.max(4, Math.sqrt(equipmentCount) * 1.2));
```

### JSX complet carte
```tsx
<ComposableMap
  projection="geoMercator"
  projectionConfig={projection}
  className="h-[340px] w-full"
>
  <ZoomableGroup center={[2.5, 46.5]} zoom={1}>
    <Geographies geography={GEO_URL}>
      {({ geographies }) =>
        geographies.map(geo => (
          <Geography
            key={geo.rsmKey}
            geography={geo}
            fill="#F1F5F9"
            stroke="#E2E8F0"
            strokeWidth={0.5}
            style={{ default: { outline: 'none' }, hover: { outline: 'none' }, pressed: { outline: 'none' } }}
          />
        ))
      }
    </Geographies>
    {sites.map(site => (
      <Marker key={site.id} coordinates={[site.lng, site.lat]}>
        <Tooltip delayDuration={0}>
          <TooltipTrigger asChild>
            <circle
              r={getBubbleRadius(site.equipment_count)}
              fill={getBubbleColor(site)}
              fillOpacity={0.85}
              stroke="white"
              strokeWidth={1}
              style={{ cursor: site.imported ? 'pointer' : 'default' }}
              onClick={() => site.imported && onSiteClick(site)}
            />
          </TooltipTrigger>
          <TooltipContent>
            <p className="font-medium">{site.name}</p>
            <p className="text-xs text-muted-foreground">
              {site.imported && site.score !== null
                ? `Conformité : ${site.score}% · U1: ${site.u1} · U2: ${site.u2}`
                : 'Pas encore importé'
              }
            </p>
          </TooltipContent>
        </Tooltip>
      </Marker>
    ))}
  </ZoomableGroup>
</ComposableMap>
```

### Fallback si react-simple-maps indisponible
SVG inline avec viewBox="0 0 600 700" (contour France simplifié)
et `<circle>` à la position (x, y) calculée via une projection linéaire des lat/lng.

---

## 8. MOCK DATA PATTERN

```tsx
// Toujours en haut du composant, avant le JSX
const MOCK_DATA = {
  sites: [...],        // données simulées réalistes
  monthlyObs: [...],   // 12 mois d'observations
  categoryDist: [...], // répartition par catégorie
};

// Sélection données réelles vs mock
const displayData = realData.sites.length >= 3 ? realData : MOCK_DATA;
const isMock = realData.sites.length < 3;

// Banner mock en bas de page (dismissible)
{isMock && !mockDismissed && (
  <Alert className="mt-4 border-[#E2E8F0] bg-[#F8F9FB]">
    <AlertDescription className="text-xs text-[#64748B] flex items-center justify-between">
      <span>Mode démo — certaines données sont simulées. Importez des rapports VGP pour voir vos données réelles.</span>
      <Button variant="ghost" size="sm" className="h-6 text-xs" onClick={() => setMockDismissed(true)}>Fermer</Button>
    </AlertDescription>
  </Alert>
)}
```

---

## 9. RÈGLES ABSOLUES

| Règle | Interdit | Obligatoire |
|---|---|---||
| Composants UI | Tout autre library | `shadcn/ui` exclusivement |
| Icônes | Heroicons, FA, etc. | `lucide-react` exclusivement |
| Graphiques | D3, Highcharts, etc. | `recharts` via `ChartContainer` |
| Langue UI | Anglais | 100% français |
| Routing | react-router-dom | `@tanstack/react-router` |
| Nouvelles libs | Sans demande explicite | Vérifier package.json d'abord |
| Dark mode | — | Ne pas implémenter |
| Fond de page | Blanc pur | `bg-[#F8F9FB]` |
| Breakpoint min | Mobile (<1024px) | 1280px desktop |
| CSS custom | Si shadcn suffit | Tailwind classes |

### Responsive
- Grilles KPI : `grid-cols-2 sm:grid-cols-4 xl:grid-cols-5`
- Graphiques côte à côte : `grid-cols-1 lg:grid-cols-3`
- Carte + top5 : `grid-cols-1 lg:grid-cols-2`
- Tables : scroll horizontal sur mobile (`overflow-x-auto`)

### Accessibilité
- Tous les éléments interactifs : `cursor-pointer`
- Icônes décoratives : `aria-hidden="true"`
- Boutons icônes seuls : `aria-label` obligatoire
- Contrastes : vérifier 4.5:1 sur tous les textes

### Performance
- `useMemo` pour les calculs d'agrégation (totaux, moyennes)
- `useCallback` pour les handlers de graphiques
- Listes > 50 items : virtualisées ou paginées
