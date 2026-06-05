# Avoid Over-Engineering

Solve the problem in front of you, not the one you imagine might appear next year. Add abstraction only when real pressure makes the current concrete solution painful.

## Use This Rule When

- Designing module boundaries.
- Deciding whether to add service, repository, mapper, or factory layers.
- Generalizing a solution after seeing one implementation.

## Prefer

- Concrete types and direct return shapes.
- Straightforward modules with a small public API.
- Repeating simple code briefly until the common shape is proven.
- Incremental extraction after duplication becomes real and stable.

## Avoid

- Repository and service stacks for simple CRUD or glue code.
- DTO and entity splits with no real boundary or transformation need.
- Generic abstractions before multiple implementations exist.
- Configuration knobs that nobody is using yet.

## Add a Layer Only If

- Multiple current call sites need the same behavior.
- The abstraction removes repeated changes, not just repeated lines.
- There is a real boundary with an unstable dependency or external system.
- The new layer has a clear name and responsibility.

## Review Checklist

- What concrete pain does this abstraction remove today?
- If the abstraction disappeared, what would get worse right now?
- Are we modeling a real boundary, or inventing ceremony?
- Can the caller get the exact data shape it needs directly?

## Example

**Bad:**

```ts
interface IRepository<T> {
  findById(id: string): Promise<T | null>;
}

class UserEntity {
  constructor(
    public id: string,
    public name: string,
  ) {}
}

class UserDTO {
  constructor(
    public userId: string,
    public userName: string,
  ) {}
}

class UserMapper {
  static toDTO(entity: UserEntity): UserDTO {
    return new UserDTO(entity.id, entity.name);
  }
}

class UserRepository implements IRepository<UserEntity> {
  async findById(id: string): Promise<UserEntity | null> {
    const data = await db.query(`SELECT * FROM users WHERE id = ${id}`);
    return data ? new UserEntity(data.id, data.name) : null;
  }
}

class UserService {
  constructor(private userRepo: IRepository<UserEntity>) {}

  async getUser(id: string): Promise<UserDTO | null> {
    const user = await this.userRepo.findById(id);
    return user ? UserMapper.toDTO(user) : null;
  }
}
```

**Good:**

```ts
interface User {
  id: string;
  name: string;
}

export async function getUserById(id: string): Promise<User | null> {
  const data = await db.query(`SELECT * FROM users WHERE id = ${id}`);

  if (!data) return null;

  return {
    id: data.id,
    name: data.name,
  };
}
```
