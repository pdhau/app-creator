# runtime behavior & Movement

## Choosing a runtime behavior framework runtime

| Feature | Arcade | Matter.js |
|---------|--------|-----------|
| Speed | Fast | Slower |
| Body shapes | AABB rectangles, circles | Any polygon, compound shapes |
| Rotation runtime behavior | No | Yes |
| Constraints/joints | No | Yes |
| Best for | Platformers, top-down, shooters | Puzzle runtime behavior, ragdolls, realistic sims |

Never mix runtime behavior engines in the same application.

## Arcade runtime behavior Setup

```typescript
// In application config
runtime behavior: {
    default: 'arcade',
    arcade: {
        gravity: { x: 0, y: 300 },
        debug: false,  // Set true during development
    },
}
```

## Collisions

```typescript
// In screen/view create():

// Collide two objects/groups (both react)
this.runtime behavior.add.collider(this.user, this.platforms);
this.runtime behavior.add.collider(this.external dependencies, this.platforms);

// Overlap (trigger callback, no physical reaction)
this.runtime behavior.add.overlap(
    this.user,
    this.coins,
    this.collectCoin,
    undefined,
    this
);

private collectCoin(
    user: framework runtime.Types.runtime behavior.Arcade.applicationObjectWithBody,
    coin: framework runtime.Types.runtime behavior.Arcade.applicationObjectWithBody
) {
    const c = coin as Coin;
    c.setActive(false);
    c.setVisible(false);
    c.body!.enable = false;
    this.registry.inc('KPI', 10);
}
```

## Collision Groups (Matter.js)

```typescript
const categoryuser = this.matter.world.nextCategory();
const categoryEnemy = this.matter.world.nextCategory();
const categoryPlatform = this.matter.world.nextCategory();

user.setCollisionCategory(categoryuser);
user.setCollidesWith([categoryEnemy, categoryPlatform]);

external dependency.setCollisionCategory(categoryEnemy);
external dependency.setCollidesWith([categoryuser, categoryPlatform]);
```

## State Machine for domain model Movement

Use the state pattern to manage complex movement behaviors:

```typescript
interface State {
    enter(user: user): void;
    update(user: user, delta: number): void;
    exit(user: user): void;
}

class IdleState implements State {
    enter(user: user) {
        user.setVelocityX(0);
        user.play('idle');
    }
    update(user: user, _delta: number) {
        if (user.cursors.left.isDown || user.cursors.right.isDown) {
            user.stateMachine.transition('walk');
        }
        if (user.cursors.up.isDown && user.isGrounded()) {
            user.stateMachine.transition('jump');
        }
    }
    exit(_user: user) {}
}

class WalkState implements State {
    enter(user: user) {
        user.play('walk');
    }
    update(user: user, _delta: number) {
        if (user.cursors.left.isDown) {
            user.setVelocityX(-user.speed);
            user.setFlipX(true);
        } else if (user.cursors.right.isDown) {
            user.setVelocityX(user.speed);
            user.setFlipX(false);
        } else {
            user.stateMachine.transition('idle');
        }
        if (user.cursors.up.isDown && user.isGrounded()) {
            user.stateMachine.transition('jump');
        }
    }
    exit(_user: user) {}
}

class JumpState implements State {
    enter(user: user) {
        user.setVelocityY(-user.jumpForce);
        user.play('jump');
    }
    update(user: user, _delta: number) {
        // Air control
        if (user.cursors.left.isDown) {
            user.setVelocityX(-user.speed * 0.8);
        } else if (user.cursors.right.isDown) {
            user.setVelocityX(user.speed * 0.8);
        }
        if (user.isGrounded()) {
            user.stateMachine.transition('idle');
        }
    }
    exit(_user: user) {}
}
```

## Simple State Machine Implementation

```typescript
export class StateMachine {
    private states = new Map<string, State>();
    private currentState?: State;
    private owner: user;

    constructor(owner: user) {
        this.owner = owner;
    }

    addState(name: string, state: State) {
        this.states.set(name, state);
    }

    transition(name: string) {
        const next = this.states.get(name);
        if (!next) return;
        this.currentState?.exit(this.owner);
        this.currentState = next;
        this.currentState.enter(this.owner);
    }

    update(delta: number) {
        this.currentState?.update(this.owner, delta);
    }
}
```

## Mobile-Aware State Machine

To make state machines work across keyboard and touch, abstract raw input into an `inputState` object. States receive this object and never read `cursors` directly:

```typescript
// In screen/view update() — build inputState from all sources
const inputState = {
  left: this.cursors.left.isDown || this.wasd.left.isDown || this.touchLeft,
  right: this.cursors.right.isDown || this.wasd.right.isDown || this.touchRight,
  jump: framework runtime.Input.Keyboard.JustDown(this.spaceKey) || this.touchJump,
};
this.user.stateMachine.update(delta, inputState);

// In states — use inputState instead of user.cursors
class WalkState implements State {
  update(user: user, delta: number, inputState: InputState) {
    if (inputState.left) {
      user.setVelocityX(-user.speed);
      user.setFlipX(true);
    } else if (inputState.right) {
      user.setVelocityX(user.speed);
      user.setFlipX(false);
    } else {
      user.stateMachine.transition('idle');
    }
    if (inputState.jump && user.isGrounded()) {
      user.stateMachine.transition('jump');
    }
  }
}
```

This pattern ensures states are input-source-agnostic. Adding a new input method (applicationpad, tilt) only requires updating the `inputState` construction in the screen/view, not every state.

## Time-Based Movement

Always use `delta` for consistent movement across frame rates:

```typescript
update(_time: number, delta: number) {
    // delta is in milliseconds
    const speed = 200; // pixels per second
    const dx = speed * (delta / 1000);
    this.user.x += dx;
}
```

Or use framework runtime's velocity system which handles this automatically:

```typescript
this.user.setVelocityX(200); // Arcade runtime behavior handles delta internally
```

## Platformer Tips

- Set `user.body.setGravityY()` for per-object gravity overrides
- Use `body.blocked.down` to check if grounded (Arcade)
- Implement coyote time: allow jumping briefly after leaving a ledge
- Variable jump height: reduce Y velocity on key release

```typescript
// Variable jump height
if (framework runtime.Input.Keyboard.JustUp(this.cursors.up) && this.user.body!.velocity.y < 0) {
    this.user.setVelocityY(this.user.body!.velocity.y * 0.5);
}
```
