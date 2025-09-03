# User Interface Integration & Agent Interaction

## 🎯 UI Integration Overview

The Travel Planner's frontend provides an **interactive graph-based interface** that visualizes trip components and allows granular modification of agent-generated content. Users can directly interact with individual agents through intuitive visual elements.

## 🎨 Graph-Based Trip Visualization

### 1. Interactive Trip Graph Components

#### Node Types & Visual Representation
```typescript
interface TripNode {
  id: string;
  type: 'flight' | 'hotel' | 'activity' | 'transport' | 'meal' | 'free_time';
  agentResponsible: AgentType;
  status: 'planning' | 'confirmed' | 'booked' | 'completed';
  data: NodeData;
  position: { x: number; y: number };
  dependencies: string[];
  modifiable: boolean;
}

// Visual styling by agent type
const agentColors = {
  planning: '#6366f1',      // Indigo - strategic planning
  transport: '#3b82f6',     // Blue - movement/travel
  accommodation: '#10b981', // Green - stability/rest
  activity: '#f59e0b',      // Amber - experiences/fun
  budget: '#ef4444',        // Red - costs/finances
  location: '#8b5cf6',      // Purple - places/geography
  weather: '#06b6d4'        // Cyan - environmental
};
```

#### Edge Types & Relationships
```typescript
interface TripEdge {
  id: string;
  source: string;
  target: string;
  type: 'dependency' | 'sequence' | 'preference' | 'constraint';
  strength: 'required' | 'recommended' | 'optional';
  data: {
    description: string;
    impact: 'high' | 'medium' | 'low';
    modifiable: boolean;
  };
}

// Visual styling by relationship type
const edgeStyles = {
  dependency: { color: '#ef4444', style: 'solid', width: 3 },    // Critical path
  sequence: { color: '#6b7280', style: 'solid', width: 2 },      // Time order
  preference: { color: '#10b981', style: 'dashed', width: 2 },   // User preference
  constraint: { color: '#f59e0b', style: 'dotted', width: 2 }    // External constraint
};
```

### 2. Real-Time Agent Status Display

#### Agent Activity Monitor
```typescript
interface AgentStatus {
  agentId: string;
  agentType: AgentType;
  status: 'idle' | 'processing' | 'waiting' | 'error';
  currentTask?: {
    taskId: string;
    description: string;
    progress: number;
    estimatedCompletion: Date;
  };
  queuedTasks: number;
  lastActivity: Date;
  performance: {
    averageResponseTime: number;
    successRate: number;
    cost: number;
  };
}

// Real-time status component
const AgentStatusWidget: React.FC<{ agentStatus: AgentStatus }> = ({ agentStatus }) => {
  return (
    <div className={`agent-status ${agentStatus.status}`}>
      <div className="agent-header">
        <AgentIcon type={agentStatus.agentType} />
        <span className="agent-name">{agentStatus.agentType}</span>
        <StatusIndicator status={agentStatus.status} />
      </div>
      
      {agentStatus.currentTask && (
        <div className="current-task">
          <ProgressBar progress={agentStatus.currentTask.progress} />
          <span className="task-description">
            {agentStatus.currentTask.description}
          </span>
        </div>
      )}
      
      <div className="agent-metrics">
        <Metric label="Response Time" value={`${agentStatus.performance.averageResponseTime}s`} />
        <Metric label="Success Rate" value={`${agentStatus.performance.successRate}%`} />
        <Metric label="Cost" value={`$${agentStatus.performance.cost.toFixed(3)}`} />
      </div>
    </div>
  );
};
```

## 🔧 Node-Based Editing Interface

### 1. Component Modification System

#### Node Click Interaction Flow
```typescript
const handleNodeClick = async (nodeId: string, nodeType: string) => {
  // 1. Identify responsible agent
  const responsibleAgent = getAgentForNodeType(nodeType);
  
  // 2. Show loading state
  setNodeEditingState(nodeId, 'loading');
  
  // 3. Activate specific agent for modification
  const modificationOptions = await activateAgentForModification({
    agentType: responsibleAgent,
    nodeId: nodeId,
    currentData: getNodeData(nodeId),
    tripContext: getCurrentTripContext(),
    userPreferences: getUserPreferences()
  });
  
  // 4. Display modification interface
  showModificationPanel({
    nodeId,
    nodeType,
    options: modificationOptions,
    onSelect: handleOptionSelection,
    onCustomize: handleCustomModification
  });
};

const handleOptionSelection = async (nodeId: string, selectedOption: any) => {
  // 1. Update node with new selection
  updateNode(nodeId, selectedOption);
  
  // 2. Trigger affected agents for recalculation
  const affectedComponents = calculateAffectedComponents(nodeId, selectedOption);
  
  // 3. Show impact preview
  const impactPreview = await calculateImpactPreview(affectedComponents);
  showImpactPreview(impactPreview);
  
  // 4. Allow user to confirm or modify
  if (await confirmChanges(impactPreview)) {
    await applyChangesWithAgentCoordination(affectedComponents);
  }
};
```

#### Dynamic Modification Panels
```typescript
interface ModificationPanel {
  nodeType: string;
  availableOptions: Option[];
  customizationFields: Field[];
  impactPreview: ImpactPreview;
  agentRecommendations: Recommendation[];
}

const ModificationPanel: React.FC<{ panel: ModificationPanel }> = ({ panel }) => {
  return (
    <div className="modification-panel">
      <div className="panel-header">
        <h3>Modify {panel.nodeType}</h3>
        <AgentBadge agentType={getResponsibleAgent(panel.nodeType)} />
      </div>
      
      <div className="options-section">
        <h4>Available Options</h4>
        {panel.availableOptions.map(option => (
          <OptionCard 
            key={option.id}
            option={option}
            onSelect={() => handleOptionSelect(option)}
            recommended={panel.agentRecommendations.includes(option.id)}
          />
        ))}
      </div>
      
      <div className="customization-section">
        <h4>Custom Requirements</h4>
        {panel.customizationFields.map(field => (
          <CustomField 
            key={field.name}
            field={field}
            onChange={(value) => handleFieldChange(field.name, value)}
          />
        ))}
      </div>
      
      <div className="impact-preview">
        <h4>Impact Preview</h4>
        <ImpactVisualization impact={panel.impactPreview} />
      </div>
    </div>
  );
};
```

### 2. Progressive Enhancement Patterns

#### Intelligent Suggestions
```typescript
class IntelligentSuggestionEngine {
  async generateSuggestions(
    nodeId: string, 
    userAction: UserAction, 
    tripContext: TripContext
  ): Promise<Suggestion[]> {
    
    // Analyze user's modification pattern
    const userPattern = this.analyzeUserPattern(userAction, tripContext);
    
    // Get agent-specific suggestions
    const agentSuggestions = await Promise.all([
      this.getBudgetOptimizations(nodeId, tripContext),
      this.getExperienceEnhancements(nodeId, tripContext),
      this.getLogisticalImprovements(nodeId, tripContext),
      this.getPersonalizationOptions(nodeId, userPattern)
    ]);
    
    // Rank and filter suggestions
    const rankedSuggestions = this.rankSuggestions(
      agentSuggestions.flat(),
      userPattern,
      tripContext
    );
    
    return rankedSuggestions.slice(0, 5); // Top 5 suggestions
  }
  
  private async getBudgetOptimizations(nodeId: string, context: TripContext) {
    return await this.callAgent('budget', {
      action: 'suggest_optimizations',
      nodeId: nodeId,
      currentBudget: context.budget,
      flexibility: context.userPreferences.budgetFlexibility
    });
  }
  
  private async getExperienceEnhancements(nodeId: string, context: TripContext) {
    return await this.callAgent('activity', {
      action: 'suggest_enhancements',
      nodeId: nodeId,
      interests: context.userPreferences.interests,
      timeAvailable: context.schedule.flexibility
    });
  }
}
```

#### Context-Aware Recommendations
```typescript
interface SmartRecommendation {
  id: string;
  type: 'upgrade' | 'alternative' | 'addition' | 'optimization';
  agentSource: AgentType;
  title: string;
  description: string;
  impact: {
    cost: number;
    time: number;
    experience: number;
    logistics: number;
  };
  confidence: number;
  userFit: number;
}

const SmartRecommendationCard: React.FC<{ 
  recommendation: SmartRecommendation;
  onApply: (rec: SmartRecommendation) => void;
}> = ({ recommendation, onApply }) => {
  return (
    <div className="recommendation-card">
      <div className="rec-header">
        <RecommendationType type={recommendation.type} />
        <AgentBadge agentType={recommendation.agentSource} />
        <ConfidenceScore score={recommendation.confidence} />
      </div>
      
      <div className="rec-content">
        <h4>{recommendation.title}</h4>
        <p>{recommendation.description}</p>
      </div>
      
      <div className="rec-impact">
        <ImpactIndicator 
          label="Cost" 
          value={recommendation.impact.cost} 
          type="cost"
        />
        <ImpactIndicator 
          label="Time" 
          value={recommendation.impact.time} 
          type="time"
        />
        <ImpactIndicator 
          label="Experience" 
          value={recommendation.impact.experience} 
          type="experience"
        />
      </div>
      
      <div className="rec-actions">
        <Button 
          variant="primary" 
          onClick={() => onApply(recommendation)}
        >
          Apply Recommendation
        </Button>
        <Button 
          variant="secondary" 
          onClick={() => showMoreDetails(recommendation)}
        >
          More Details
        </Button>
      </div>
    </div>
  );
};
```

## 🔄 Real-Time Agent Coordination UI

### 1. Live Planning Dashboard

#### Multi-Agent Activity Visualization
```typescript
const LivePlanningDashboard: React.FC = () => {
  const [agentStates, setAgentStates] = useState<AgentState[]>([]);
  const [coordinationEvents, setCoordinationEvents] = useState<CoordinationEvent[]>([]);
  
  useEffect(() => {
    // WebSocket connection for real-time updates
    const ws = new WebSocket(WS_AGENT_COORDINATION_URL);
    
    ws.onmessage = (event) => {
      const update = JSON.parse(event.data);
      
      switch (update.type) {
        case 'agent_status_update':
          updateAgentState(update.agentId, update.status);
          break;
        case 'coordination_event':
          addCoordinationEvent(update.event);
          break;
        case 'planning_progress':
          updatePlanningProgress(update.progress);
          break;
      }
    };
    
    return () => ws.close();
  }, []);
  
  return (
    <div className="live-planning-dashboard">
      <div className="dashboard-header">
        <h2>Live Planning Progress</h2>
        <PlanningProgress progress={planningProgress} />
      </div>
      
      <div className="agent-grid">
        {agentStates.map(agent => (
          <AgentStatusCard 
            key={agent.id}
            agent={agent}
            onInterrupt={() => handleAgentInterrupt(agent.id)}
            onPrioritize={() => handleAgentPrioritize(agent.id)}
          />
        ))}
      </div>
      
      <div className="coordination-timeline">
        <h3>Agent Coordination Timeline</h3>
        <Timeline events={coordinationEvents} />
      </div>
    </div>
  );
};
```

#### User Intervention Mechanisms
```typescript
interface UserIntervention {
  type: 'pause_agent' | 'redirect_agent' | 'provide_feedback' | 'emergency_stop';
  agentId: string;
  reason: string;
  instructions?: string;
  priority: 'low' | 'medium' | 'high' | 'critical';
}

const InterventionPanel: React.FC<{ 
  agentId: string;
  onIntervention: (intervention: UserIntervention) => void;
}> = ({ agentId, onIntervention }) => {
  return (
    <div className="intervention-panel">
      <h4>Intervene with Agent</h4>
      
      <div className="intervention-options">
        <Button
          className="pause-btn"
          onClick={() => onIntervention({
            type: 'pause_agent',
            agentId,
            reason: 'User wants to review progress',
            priority: 'medium'
          })}
        >
          Pause Agent
        </Button>
        
        <Button
          className="redirect-btn"
          onClick={() => showRedirectDialog(agentId)}
        >
          Change Direction
        </Button>
        
        <Button
          className="feedback-btn"
          onClick={() => showFeedbackDialog(agentId)}
        >
          Provide Feedback
        </Button>
        
        <Button
          className="emergency-btn"
          onClick={() => onIntervention({
            type: 'emergency_stop',
            agentId,
            reason: 'User emergency stop',
            priority: 'critical'
          })}
        >
          Emergency Stop
        </Button>
      </div>
    </div>
  );
};
```

### 2. Conflict Resolution Interface

#### Visual Conflict Display
```typescript
interface ConflictVisualization {
  conflictId: string;
  type: 'budget_overflow' | 'scheduling_conflict' | 'preference_mismatch' | 'availability_issue';
  affectedNodes: string[];
  conflictingAgents: AgentType[];
  severity: 'low' | 'medium' | 'high' | 'critical';
  autoResolutionAvailable: boolean;
  userResolutionRequired: boolean;
}

const ConflictResolutionPanel: React.FC<{ 
  conflict: ConflictVisualization;
  onResolve: (resolution: ConflictResolution) => void;
}> = ({ conflict, onResolve }) => {
  const [selectedResolution, setSelectedResolution] = useState<string | null>(null);
  
  return (
    <div className={`conflict-panel severity-${conflict.severity}`}>
      <div className="conflict-header">
        <ConflictIcon type={conflict.type} severity={conflict.severity} />
        <h3>Conflict Detected</h3>
        <SeverityBadge severity={conflict.severity} />
      </div>
      
      <div className="conflict-description">
        <ConflictDescription conflict={conflict} />
        <AffectedNodesHighlight nodeIds={conflict.affectedNodes} />
      </div>
      
      <div className="resolution-options">
        <h4>Resolution Options</h4>
        {getResolutionOptions(conflict).map(option => (
          <ResolutionOption
            key={option.id}
            option={option}
            selected={selectedResolution === option.id}
            onSelect={() => setSelectedResolution(option.id)}
          />
        ))}
      </div>
      
      <div className="resolution-actions">
        {conflict.autoResolutionAvailable && (
          <Button
            variant="secondary"
            onClick={() => onResolve({ type: 'auto', conflictId: conflict.conflictId })}
          >
            Auto-Resolve
          </Button>
        )}
        
        <Button
          variant="primary"
          disabled={!selectedResolution}
          onClick={() => onResolve({ 
            type: 'manual', 
            conflictId: conflict.conflictId,
            selectedOption: selectedResolution 
          })}
        >
          Apply Resolution
        </Button>
      </div>
    </div>
  );
};
```

## 📱 Responsive Agent Interaction

### 1. Mobile-Optimized Agent Interface

#### Touch-Friendly Node Interaction
```typescript
const MobileNodeInterface: React.FC<{ node: TripNode }> = ({ node }) => {
  const [gestureState, setGestureState] = useState<GestureState>('idle');
  
  const handleTouch = useTouchGesture({
    onTap: () => handleNodeTap(node.id),
    onLongPress: () => showContextMenu(node.id),
    onSwipeLeft: () => showAlternatives(node.id),
    onSwipeRight: () => showDetails(node.id),
    onPinch: (scale) => handleNodeResize(node.id, scale)
  });
  
  return (
    <div 
      className={`mobile-node ${node.type} ${gestureState}`}
      {...handleTouch}
    >
      <div className="node-icon">
        <NodeIcon type={node.type} agentType={node.agentResponsible} />
      </div>
      
      <div className="node-info">
        <span className="node-title">{node.data.title}</span>
        <span className="agent-badge">{node.agentResponsible}</span>
      </div>
      
      <div className="node-status">
        <StatusIndicator status={node.status} />
      </div>
      
      {gestureState === 'long_press' && (
        <MobileContextMenu nodeId={node.id} />
      )}
    </div>
  );
};
```

#### Simplified Agent Controls
```typescript
const MobileAgentControls: React.FC = () => {
  return (
    <div className="mobile-agent-controls">
      <div className="quick-actions">
        <QuickActionButton
          icon="pause"
          label="Pause All"
          action={() => pauseAllAgents()}
        />
        <QuickActionButton
          icon="refresh"
          label="Refresh"
          action={() => refreshPlan()}
        />
        <QuickActionButton
          icon="optimize"
          label="Optimize"
          action={() => optimizePlan()}
        />
      </div>
      
      <div className="agent-status-compact">
        {agents.map(agent => (
          <CompactAgentStatus 
            key={agent.id}
            agent={agent}
            onClick={() => showAgentDetails(agent.id)}
          />
        ))}
      </div>
    </div>
  );
};
```

### 2. Voice Interface Integration

#### Voice Commands for Agent Control
```typescript
class VoiceAgentInterface {
  private speechRecognition: SpeechRecognition;
  private commandPatterns: CommandPattern[];
  
  constructor() {
    this.speechRecognition = new webkitSpeechRecognition();
    this.commandPatterns = [
      {
        pattern: /change (hotel|flight|activity) to (.+)/i,
        handler: this.handleChangeCommand
      },
      {
        pattern: /optimize for (budget|time|experience)/i,
        handler: this.handleOptimizeCommand
      },
      {
        pattern: /show me alternatives for (.+)/i,
        handler: this.handleAlternativesCommand
      },
      {
        pattern: /pause (all agents|(.+) agent)/i,
        handler: this.handlePauseCommand
      }
    ];
  }
  
  startListening() {
    this.speechRecognition.onresult = (event) => {
      const command = event.results[0][0].transcript;
      this.processVoiceCommand(command);
    };
    
    this.speechRecognition.start();
  }
  
  private processVoiceCommand(command: string) {
    for (const pattern of this.commandPatterns) {
      const match = command.match(pattern.pattern);
      if (match) {
        pattern.handler(match);
        return;
      }
    }
    
    // If no pattern matches, use natural language processing
    this.handleNaturalLanguageCommand(command);
  }
  
  private async handleNaturalLanguageCommand(command: string) {
    const intent = await this.parseNaturalLanguageIntent(command);
    
    switch (intent.type) {
      case 'modify_component':
        await this.executeModification(intent.parameters);
        break;
      case 'request_information':
        await this.provideInformation(intent.parameters);
        break;
      case 'agent_control':
        await this.controlAgent(intent.parameters);
        break;
    }
  }
}
```

## 🔧 Environment Configuration for UI Integration

### .env Configuration for UI Settings
```env
# UI Integration Settings
GRAPH_VISUALIZATION_ENABLED=true
REAL_TIME_UPDATES_ENABLED=true
VOICE_INTERFACE_ENABLED=true
MOBILE_OPTIMIZED=true

# WebSocket Configuration
WS_AGENT_COORDINATION_URL=ws://localhost:8080/agent-coordination
WS_RECONNECT_ATTEMPTS=5
WS_HEARTBEAT_INTERVAL=30000

# Graph Display Settings
MAX_NODES_DISPLAY=50
NODE_ANIMATION_ENABLED=true
EDGE_ANIMATION_ENABLED=true
AUTO_LAYOUT_ALGORITHM=force_directed

# Agent Status Display
AGENT_STATUS_REFRESH_INTERVAL=2000
SHOW_PERFORMANCE_METRICS=true
SHOW_COST_INFORMATION=true
COMPACT_MODE_THRESHOLD=768

# Interaction Settings
NODE_CLICK_DELAY=300
LONG_PRESS_DURATION=800
GESTURE_SENSITIVITY=medium
AUTO_SAVE_INTERVAL=10000

# Conflict Resolution
CONFLICT_AUTO_HIGHLIGHT=true
CONFLICT_NOTIFICATION_ENABLED=true
RESOLUTION_TIMEOUT_SECONDS=300

# Mobile Settings
MOBILE_BREAKPOINT=768
TOUCH_TARGET_SIZE=44
SWIPE_THRESHOLD=50
PINCH_SENSITIVITY=0.1

# Voice Interface
VOICE_COMMANDS_ENABLED=true
SPEECH_RECOGNITION_LANGUAGE=en-US
VOICE_FEEDBACK_ENABLED=true
COMMAND_CONFIDENCE_THRESHOLD=0.8

# Performance
UI_UPDATE_THROTTLE=100
ANIMATION_PERFORMANCE_MODE=auto
MEMORY_USAGE_LIMIT=50MB
```

### Component-Specific Configuration
```yaml
ui_components:
  trip_graph:
    max_zoom: 3.0
    min_zoom: 0.1
    pan_enabled: true
    selection_enabled: true
    
  agent_dashboard:
    refresh_rate: 2000
    max_history_items: 100
    auto_scroll: true
    
  modification_panel:
    max_options_display: 10
    auto_preview: true
    confirmation_required: true
    
  conflict_resolution:
    auto_detect: true
    severity_colors:
      low: "#10b981"
      medium: "#f59e0b" 
      high: "#ef4444"
      critical: "#7c2d12"
```

---

**Key UI Integration Principles:**
1. **Visual Clarity**: Clear representation of agent responsibilities and trip structure
2. **Intuitive Interaction**: Natural and discoverable ways to modify trip components
3. **Real-Time Feedback**: Immediate updates on agent status and coordination
4. **Progressive Enhancement**: Advanced features available without overwhelming basic users
5. **Multi-Modal Access**: Support for touch, voice, and traditional input methods