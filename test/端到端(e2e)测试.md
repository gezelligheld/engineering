单元测试和集成测试在部署到生产时，对应用整体覆盖的能力有限。端到端（E2E）测试关注当用户实际使用你的应用时会发生什么。

当端到端测试在 CI/CD 管道中运行时，它们会在无头浏览器（puppeteer，即不带界面的浏览器）中运行，通常借助Cypress、Playwright等端到端解决方案。

至此，测试金字塔的三层单元测试->集成测试->端到端测试介绍完毕。