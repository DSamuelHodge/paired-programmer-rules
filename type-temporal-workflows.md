# Type System and Function Composition Guide for Temporal Workflows

## Core Type Definitions

### Base Types and Interfaces
```typescript
// Base Activity Input/Output
interface BaseActivityInput {
  requestId: string;
  tenantId: string;
  timestamp: number;
  metadata: Record<string, unknown>;
}

interface BaseActivityOutput {
  status: 'success' | 'failure';
  timestamp: number;
  metadata: Record<string, unknown>;
}

// Base Workflow Input/Output
interface BaseWorkflowInput extends BaseActivityInput {
  workflowId: string;
  parentWorkflowId?: string;
}

interface BaseWorkflowOutput extends BaseActivityOutput {
  workflowId: string;
  activities: ActivityResult[];
}

// Activity Result Tracking
interface ActivityResult {
  activityId: string;
  status: ActivityStatus;
  startTime: number;
  endTime: number;
  attempts: number;
}

type ActivityStatus = 
  | 'SCHEDULED'
  | 'RUNNING'
  | 'COMPLETED'
  | 'FAILED'
  | 'CANCELLED';
```

### Function Signatures
```typescript
// Activity Function Types
type ActivityFn<TInput extends BaseActivityInput, TOutput extends BaseActivityOutput> = 
  (input: TInput) => Promise<TOutput>;

// Workflow Function Types
type WorkflowFn<TInput extends BaseWorkflowInput, TOutput extends BaseWorkflowOutput> = 
  (input: TInput) => Promise<TOutput>;

// Compositional Function Types
type ComposableFn<TInput, TOutput> = 
  (input: TInput) => Promise<TOutput>;

type PipelineFn<TInput, TIntermediate, TOutput> = 
  (input: TInput) => Promise<ComposableFn<TIntermediate, TOutput>>;
```

## Function Composition Patterns

### Activity Composition
```typescript
// Activity Composer
class ActivityComposer<TInput extends BaseActivityInput, TOutput extends BaseActivityOutput> {
  constructor(
    private readonly activities: ActivityFn<any, any>[],
    private readonly errorHandler: ErrorHandler
  ) {}

  compose(): ActivityFn<TInput, TOutput> {
    return async (input: TInput): Promise<TOutput> => {
      let result: any = input;
      
      for (const activity of this.activities) {
        try {
          result = await activity(result);
        } catch (error) {
          result = await this.errorHandler.handle(error, activity, result);
        }
      }
      
      return result as TOutput;
    };
  }
}

// Usage Example
const processDataActivity = new ActivityComposer([
  validateInput,
  transformData,
  enrichData,
  validateOutput
], new ErrorHandler()).compose();
```

### Workflow Composition
```typescript
// Workflow Composer
class WorkflowComposer<TInput extends BaseWorkflowInput, TOutput extends BaseWorkflowOutput> {
  private activities: Array<ActivityFn<any, any>> = [];
  private compensations: Map<string, ActivityFn<any, any>> = new Map();

  addActivity<T extends BaseActivityInput, U extends BaseActivityOutput>(
    activity: ActivityFn<T, U>,
    compensation?: ActivityFn<U, T>
  ): this {
    this.activities.push(activity);
    if (compensation) {
      this.compensations.set(activity.name, compensation);
    }
    return this;
  }

  compose(): WorkflowFn<TInput, TOutput> {
    return async (input: TInput): Promise<TOutput> => {
      const results: ActivityResult[] = [];
      let currentData: any = input;

      for (const activity of this.activities) {
        const startTime = Date.now();
        try {
          currentData = await activity(currentData);
          results.push({
            activityId: activity.name,
            status: 'COMPLETED',
            startTime,
            endTime: Date.now(),
            attempts: 1
          });
        } catch (error) {
          await this.compensate(results, currentData);
          throw error;
        }
      }

      return {
        ...currentData,
        workflowId: input.workflowId,
        activities: results
      } as TOutput;
    };
  }

  private async compensate(
    results: ActivityResult[],
    lastSuccessfulData: any
  ): Promise<void> {
    for (const result of [...results].reverse()) {
      const compensation = this.compensations.get(result.activityId);
      if (compensation) {
        await compensation(lastSuccessfulData);
      }
    }
  }
}
```

## Convention-based Configuration

### Activity Decorators
```typescript
// Activity Configuration Decorator
function ActivityConfig<T extends BaseActivityInput, U extends BaseActivityOutput>(
  config: ActivityConfiguration
) {
  return function (
    target: ActivityFn<T, U>,
    _context: ClassDecoratorContext
  ) {
    return class extends target {
      static readonly configuration = config;
      
      async execute(input: T): Promise<U> {
        // Apply configuration
        return super.execute(input);
      }
    };
  };
}

// Usage
@ActivityConfig({
  retryPolicy: standardRetryPolicy,
  rateLimiter: standardRateLimiter,
  timeout: '5m'
})
class DataProcessingActivity implements ActivityFn<DataInput, DataOutput> {
  async execute(input: DataInput): Promise<DataOutput> {
    // Implementation
  }
}
```

### Type Guards and Validators
```typescript
// Type Guard Utilities
function isBaseActivityInput(input: unknown): input is BaseActivityInput {
  return (
    typeof input === 'object' &&
    input !== null &&
    'requestId' in input &&
    'tenantId' in input &&
    'timestamp' in input
  );
}

// Validation Decorator
function ValidateInput<T extends BaseActivityInput>() {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;
    descriptor.value = function (input: T) {
      if (!isBaseActivityInput(input)) {
        throw new ValidationError('Invalid input structure');
      }
      return originalMethod.apply(this, [input]);
    };
    return descriptor;
  };
}
```

## Error Handling Types

```typescript
// Error Types
interface WorkflowError extends Error {
  code: string;
  details: Record<string, unknown>;
  retryable: boolean;
}

class ValidationError extends Error implements WorkflowError {
  code = 'VALIDATION_ERROR';
  retryable = false;
  details: Record<string, unknown>;

  constructor(message: string, details: Record<string, unknown> = {}) {
    super(message);
    this.details = details;
  }
}

// Error Handler Type
interface ErrorHandler {
  handle(
    error: WorkflowError,
    activity: ActivityFn<any, any>,
    lastSuccessfulData: any
  ): Promise<any>;
}
```

## State Management Types

```typescript
// State Management
interface WorkflowState<T> {
  current: T;
  history: T[];
  metadata: Record<string, unknown>;
  timestamp: number;
}

class StateManager<T> {
  private state: WorkflowState<T>;

  constructor(initialState: T) {
    this.state = {
      current: initialState,
      history: [],
      metadata: {},
      timestamp: Date.now()
    };
  }

  update(newState: T): void {
    this.state.history.push(this.state.current);
    this.state.current = newState;
    this.state.timestamp = Date.now();
  }

  getCurrentState(): T {
    return this.state.current;
  }

  getHistory(): T[] {
    return this.state.history;
  }
}
```

## Usage Examples

### Creating a Type-Safe Workflow
```typescript
// Example of a type-safe workflow composition
interface ImageProcessingInput extends BaseWorkflowInput {
  imageUrl: string;
  processingOptions: {
    resize?: boolean;
    format?: 'jpg' | 'png';
    quality?: number;
  };
}

interface ImageProcessingOutput extends BaseWorkflowOutput {
  processedImageUrl: string;
  processingStats: {
    originalSize: number;
    processedSize: number;
    processingTime: number;
  };
}

const imageProcessingWorkflow = new WorkflowComposer<
  ImageProcessingInput,
  ImageProcessingOutput
>()
  .addActivity(validateImage)
  .addActivity(processImage)
  .addActivity(optimizeImage)
  .addActivity(validateOutput)
  .compose();
```

### Type-Safe Activity Chain
```typescript
// Example of chaining activities with type safety
const processDataChain = new ActivityComposer<DataInput, DataOutput>([
  {
    activity: validateData,
    compensation: revertValidation,
    retryPolicy: retryPolicies.transient
  },
  {
    activity: transformData,
    compensation: revertTransformation,
    retryPolicy: retryPolicies.persistent
  },
  {
    activity: enrichData,
    retryPolicy: retryPolicies.rateLimitBackoff
  }
]).compose();
```
