md
# 🏗️ System Architecture

## Overview

Smartgentools is a production-grade Facebook automation platform built with modern technologies and following industry best practices.

## High-Level Architecture
┌─────────────────────────────────────────────────────────────┐ │ USER INTERFACES │ ├─────────────┬──────────────────┬───────────┬────────────────┤ │ React Web │ Mobile PWA │ Chrome │ Admin Dashboard │ │ Dashboard │ (iOS/Android) │ Extension │ │ └─────┬───────┴────────┬─────────┴─────┬─────┴────────┬───────┘ │ │ │ │ └────────────────┼───────────────┴──────────────┘ │ ┌───────────▼──────────────┐ │ Express.js API Layer │ │ (REST + WebSocket) │ └────────────┬─────────────┘ │ ┌───────────────┼───────────────┐ │ │ │ ┌────▼─────┐ ┌─────▼──────┐ ┌─────▼─────┐ │ Bull │ │ MongoDB │ │ Auth & │ │ Queue │ │ Database │ │ Session │ │ (Redis) │ │ │ │ Manager │ └────┬─────┘ └────────────┘ └───────────┘ │ ┌────▼──────────────────────────────┐ │ Worker Pool (Node.js) │ │ │ │ ├─ Playwright Automation │ │ ├─ Task Executor │ │ ├─ Comment Scanner │ │ └─ Error Handler & Retry Logic │ └────┬───────────────────────────────┘ │ ┌────▼─────────────────────────────┐ │ Playwright Browsers │ │ (Headless Chrome Instances) │ └────┬─────────────────────────────┘ │ ┌────▼──────────────────────────────┐ │ Facebook.com │ │ (Browser Session - Cookie-based) │ └────────────────────────────────────┘