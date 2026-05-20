# Data Model: 007-team-05-frontend-cobertura

Este archivo define los modelos de estado principales desde la perspectiva Frontend (la interfaz hacia las vistas). Las estructuras se corresponden directamente con las respuestas de la REST API (TEAM-05).

## Store: AIChatStore (useSyncExternalStore)

Ubicación: `src/store/chat.ts`
Patrón: `useSyncExternalStore` (mismo que `src/store/signals.ts`)

```typescript
export interface ChatMessage {
  id: string;
  role: 'user' | 'assistant' | 'system';
  content: string;
  timestamp: string;
  status: 'pending' | 'success' | 'error';
  responseId?: string;
  pollingAttempts?: number;
}

export interface ScenarioAnalysisItem {
  label: string;
  description: string;
  protectionLevel: 'low' | 'medium' | 'high';
  potentialPnL: number;
}

export interface ChatState {
  ticker: string | null;
  history: ChatMessage[];
  scenarios: ScenarioAnalysisItem[];
  isAiUnavailable: boolean;

  // Actions
  setContext: (ticker: string) => void;
  addMessage: (msg: ChatMessage) => void;
  updateMessageStatus: (id: string, status: ChatMessage['status'], content?: string) => void;
  incrementPolling: (id: string) => void;
  setAiUnavailable: (state: boolean) => void;
  clearHistory: () => void;
}
```

## API Interfaces

### Institutional Analysis API
Ubicación: `src/services/institutional/institutionalApi.ts`

```typescript
export interface InstitutionalAnalysisRequest {
  ticker: string;
  period: 'intraday' | 'daily' | 'weekly' | 'monthly' | 'quarterly';
  horizon: 'short' | 'medium' | 'long';
}

export interface InstitutionalAnalysisResponse {
  request: {
    ticker: string;
    period: string;
    horizon: string;
    analysisId: string;
  };
  analysis: {
    analysisId: string;
    ticker: string;
    instrument?: string;
    strike?: number;
    period: string;
    volume: number;
    liquidity: 'low' | 'medium' | 'high';
    horizon: string;
    fundsOwnershipPct: number;
    flows: { inflows: number; outflows: number; asOf: string };
    openPositions: { count: number; notional?: number };
  };
  zones: {
    all: InstitutionalZone[];
    support: InstitutionalZone[];
    resistance: InstitutionalZone[];
  };
  trends: {
    direction: 'bullish' | 'bearish' | 'neutral';
    score: number;
    confidence: number;
    rationale: string;
    supportStrength: number;
    resistanceStrength: number;
    flowBias: number;
  };
  metrics: {
    candlesAnalyzed: number;
    zoneCount: number;
    supportZoneCount: number;
    resistanceZoneCount: number;
    averageZoneStrength: number;
    maxZoneStrength: number;
    averageZoneConfidence: number;
    sourceCount: number;
    liquidity: string;
    volume: number;
    openPositions: number;
    fundsOwnershipPct: number;
    netFlow: number;
  };
  sourceReports: InstitutionalSourceReport[];
  generatedAt: string;
}

export interface InstitutionalZone {
  type: 'support' | 'resistance';
  price: number;
  strength: number;
  accumulatedVolume: number;
  confidence: number;
  confirmingSources: number;
  touches: number;
  liquidity: 'low' | 'medium' | 'high';
  asOf: string;
  notes: string[];
}

export interface InstitutionalSourceReport {
  sourceId: string;
  kind: string;
  label: string;
  status: 'ok' | 'error' | 'cached';
  tookMs: number;
  observation?: {
    asOf: string;
    confidence: number;
    volume?: number;
    fundsOwnershipPct?: number;
    openPositions?: { count: number; notional?: number };
  };
}
```

### Regulatory Positions API
Ubicación: `src/services/institutional/institutionalApi.ts`

```typescript
export interface RegulatoryPositionsResponse {
  request: {
    ticker: string;
    period: string;
    horizon: string;
    analysisId: string;
  };
  analysis: {
    ticker: string;
    period: string;
    horizon: string;
    fundsOwnershipPct: number;
    flows: { inflows: number; outflows: number; asOf: string };
    openPositions: { count: number; notional?: number };
  };
  positions13F: Array<{
    sourceId: string;
    asOf: string;
    count: number;
    notional?: number;
    fundsOwnershipPct?: number;
    volume?: number;
    confidence: number;
  }>;
  flows: {
    inflows: number;
    outflows: number;
    netFlow: number;
    asOf: string;
  };
  sourceReports: InstitutionalSourceReport[];
  cacheHit: boolean;
  usedSourceIds: string[];
}
```

### Coverage API
Ubicación: `src/services/coverage/coverageApi.ts`

```typescript
export interface CoverageAnalyzeRequest {
  ticker: string;
  currentPrice: number;
  shares: number;
  strikes: number[];
  capital: number;
  riskTolerancePct: number;
}

export interface CoverageStrategyResult {
  strategyId: string;
  kind: 'protective_put' | 'married_put' | 'collar_put' | 'covered_straddle';
  label: string;
  shares: number;
  legs: Array<{
    side: 'long' | 'short';
    type: 'call' | 'put';
    strike: number;
    premium: number;
    expiration: string;
  }>;
  netCost: number;
  breakeven: number;
  maxProfit: number;
  maxLoss: number;
  riskProfile: 'limited' | 'unlimited';
  protectionLevel: 'low' | 'medium' | 'high';
  payoffSimulation: Array<{ price: number; pnl: number; notes?: string }>;
  riskMetrics: {
    profile: string;
    protectionLevel: string;
    netPremiumPaid: number;
    stopLossPrice?: number;
    marginRequired?: number;
  };
  alerts: Array<{
    code: string;
    severity: 'info' | 'warning' | 'critical';
    message: string;
    recommendation: string;
  }>;
  recommended: boolean;
}

export interface CoverageAnalyzeResponse {
  results: CoverageStrategyResult[];
  generatedAt: string;
}

export interface CoverageCompareRequest {
  ticker: string;
  currentPrice: number;
  shares: number;
  strikes: number[];
  capital: number;
  riskTolerancePct: number;
  horizon: 'short' | 'medium' | 'long';
}

export interface CoverageCompareResponse {
  ranking: CoverageStrategyResult[];
  recommendedKind: string;
  available: boolean;
  partialData: boolean;
  generatedAt: string;
}

export interface CoverageSimulateRequest {
  ticker: string;
  currentPrice: number;
  shares: number;
  capital: number;
  strategyKind: 'protective_put' | 'married_put' | 'collar_put' | 'covered_straddle';
  mode: 'deterministic' | 'monte_carlo' | 'backtest';
  iterations?: number;
}

export interface CoverageSimulateResponse {
  strategyKind: string;
  mode: string;
  summary: {
    expectedPnL: number;
    median: number;
    worst: number;
    best: number;
    var95: number;
    shortfall95: number;
  };
  scenarios: Array<{
    label: string;
    price: number;
    pnl: number;
    probability?: number;
  }>;
  generatedAt: string;
}
```

### AI Chat API
Ubicación: `src/services/ai/aiChatApi.ts`

```typescript
export interface AIChatRequest {
  ticker: string;
  currentPrice: number;
  zones: {
    all: InstitutionalZone[];
    support: InstitutionalZone[];
    resistance: InstitutionalZone[];
  };
  coverageStrategies?: CoverageStrategyResult[];
  question: string;
  userRole?: 'analyst' | 'risk_manager';
}

export interface AIChatResponse {
  contextId: string;
  responseId: string;
  ticker: string;
  narrative: string;
  reasoning: string[];
  scenarioAnalysis: Array<{
    label: string;
    description: string;
    protectionLevel: 'low' | 'medium' | 'high';
    potentialPnL: number;
  }>;
  recommendation: string;
  evidenceIds: string[];
  modelVersion: string;
  responseHash: string;
  ai_unavailable: boolean;
  timestamp: string;
}

export interface AIChatPollingResponse {
  status: 'pending' | 'completed';
  contextId: string;
  responseId: string;
  pollingUrl?: string;
  retryAfterSeconds?: number;
  ai_unavailable: boolean;
  timestamp: string;
}
```
