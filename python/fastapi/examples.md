Use these examples as inspiration.

```py
# store.py
from proj.projects.types import ProjectGetPayload
from proj.projects.models import Project

class ProjectStore:
    def __init__(self, db: AsyncSession):
        self.db = db

    async def get(self, payload: ProjectGetPayload) -> Project:
        stmt = select(Project).where(
            and_(
                Project.uuid == payload.uuid,
                Project.user_id = payload.user_id,
            )
        )
        res = await self.db.execute(stmt)
        project = res.scalar_one_or_none()
        return project
```
