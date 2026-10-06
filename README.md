# application-creator

Bộ skill để biến một ý tưởng sản phẩm thành ứng dụng có thể chạy, kiểm thử và bàn giao. Các thư mục cũ được giữ làm alias để không làm hỏng liên kết; nội dung bên trong đã được chuyển sang domain application development.

## Workflow

1. **Idea**: làm rõ người dùng, vấn đề, user workflow, KPI, phạm vi, rủi ro và quyết định kiến trúc trong `docs/product-plan.md`, `docs/tech.md` và ADR.
2. **Scaffold**: dùng scaffolder chính thức của framework/runtime, cài dependency, tạo `AGENTS.md`, rồi kiểm tra boot và route đầu tiên.
3. **Development**: mỗi phiên bắt đầu bằng session-start, scope triage, milestone và backlog; viết test trước, triển khai theo một lát cắt nhỏ, live-verify UI/API/accessibility/performance, rồi cập nhật state và handoff.

## Skill map

- `make-app` là alias tương thích cho product planning và application delivery.
- `app-qa`/`qa-app` là application QA với boot, workflow, API/UI, accessibility và performance.
- `app-assets`, `app-audio`, `app-3d-assets` là product assets, notifications/media và rich visual assets.
- `add-multiplayer`, `app-deploy`, `monetize-app`, `promo-video` bao phủ realtime collaboration, release, monetization và demo capture.

Các tên thư mục cũ được giữ để tương thích; không còn hướng dẫn phụ thuộc vào một runtime, toolkit, media style hay nền tảng cụ thể.
