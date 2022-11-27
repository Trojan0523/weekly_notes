# Recommended workflow

## commit Description Rule

| 本次修改             | Commit Msg           |
| -------------------- | -------------------- |
| 新功能、支持某个需求 | feat:某个需求的简介  |
| 修复某个线上 BUG     | fix:解决了什么 BUG    |
| 修改文档类的         | doc:大致更新的内容   |
| 重构某一模块         | refactor:重构的描述  |
| 优化某一模块         | perf:优化的描述      |

## WorkFlow make

1. Make changes
2. Commit those changes
3. Make sure Travis turns green
4. Bump version in package.json (TODO: using bumpp)
5. conventionalChangelog (just need to execute `pnpm run version` as well as below)
6. Commit package.json and CHANGELOG.md files
7. Tag (those Tags tagging can use auto-task completing and command line to do) (git tag -a v1.0.x -m "my version 1.4")
8. Push
9. git push origin <tag_name>
