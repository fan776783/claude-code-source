# Claude Code Spec 导航

主规格文档已拆分到 `spec/` 目录，建议从下面入口开始：

- [spec/README.md](./spec/README.md)

这套文档以**当前工作树**为准，目标是：

- 描述当前可确认的实现
- 明确标注源码仍有引用、但当前工作树缺失实现的异常项

## 推荐阅读顺序

1. [spec/README.md](./spec/README.md)
2. [spec/01-项目概览.md](./spec/01-%E9%A1%B9%E7%9B%AE%E6%A6%82%E8%A7%88.md)
3. [spec/02-启动与运行时.md](./spec/02-%E5%90%AF%E5%8A%A8%E4%B8%8E%E8%BF%90%E8%A1%8C%E6%97%B6.md)
4. [spec/03-REPL与状态管理.md](./spec/03-REPL%E4%B8%8E%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86.md)
5. [spec/04-查询执行与工具系统.md](./spec/04-%E6%9F%A5%E8%AF%A2%E6%89%A7%E8%A1%8C%E4%B8%8E%E5%B7%A5%E5%85%B7%E7%B3%BB%E7%BB%9F.md)
6. [spec/05-命令-Skills-插件.md](./spec/05-%E5%91%BD%E4%BB%A4-Skills-%E6%8F%92%E4%BB%B6.md)
7. [spec/06-MCP与远程桥接.md](./spec/06-MCP%E4%B8%8E%E8%BF%9C%E7%A8%8B%E6%A1%A5%E6%8E%A5.md)
8. [spec/07-接口与协议.md](./spec/07-%E6%8E%A5%E5%8F%A3%E4%B8%8E%E5%8D%8F%E8%AE%AE.md)
9. [spec/08-技术栈与依赖图.md](./spec/08-%E6%8A%80%E6%9C%AF%E6%A0%88%E4%B8%8E%E4%BE%9D%E8%B5%96%E5%9B%BE.md)
10. [spec/09-关键时序图.md](./spec/09-%E5%85%B3%E9%94%AE%E6%97%B6%E5%BA%8F%E5%9B%BE.md)
11. [spec/10-模块索引.md](./spec/10-%E6%A8%A1%E5%9D%97%E7%B4%A2%E5%BC%95.md)
12. [spec/11-源码文件映射表.md](./spec/11-%E6%BA%90%E7%A0%81%E6%96%87%E4%BB%B6%E6%98%A0%E5%B0%84%E8%A1%A8.md)
13. [spec/12-阅读路线-按角色.md](./spec/12-%E9%98%85%E8%AF%BB%E8%B7%AF%E7%BA%BF-%E6%8C%89%E8%A7%92%E8%89%B2.md)
14. [spec/13-关键目录树.md](./spec/13-%E5%85%B3%E9%94%AE%E7%9B%AE%E5%BD%95%E6%A0%91.md)
15. [spec/14-关键源码链接索引.md](./spec/14-%E5%85%B3%E9%94%AE%E6%BA%90%E7%A0%81%E9%93%BE%E6%8E%A5%E7%B4%A2%E5%BC%95.md)
16. [spec/15-常见问题定位手册.md](./spec/15-%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E5%AE%9A%E4%BD%8D%E6%89%8B%E5%86%8C.md)

## 10 分钟速读

如果你只想快速建立认知，优先看：

1. [spec/01-项目概览.md](./spec/01-%E9%A1%B9%E7%9B%AE%E6%A6%82%E8%A7%88.md)
2. [spec/02-启动与运行时.md](./spec/02-%E5%90%AF%E5%8A%A8%E4%B8%8E%E8%BF%90%E8%A1%8C%E6%97%B6.md)
3. [spec/03-REPL与状态管理.md](./spec/03-REPL%E4%B8%8E%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86.md)
4. [spec/04-查询执行与工具系统.md](./spec/04-%E6%9F%A5%E8%AF%A2%E6%89%A7%E8%A1%8C%E4%B8%8E%E5%B7%A5%E5%85%B7%E7%B3%BB%E7%BB%9F.md)
5. [spec/09-关键时序图.md](./spec/09-%E5%85%B3%E9%94%AE%E6%97%B6%E5%BA%8F%E5%9B%BE.md)
