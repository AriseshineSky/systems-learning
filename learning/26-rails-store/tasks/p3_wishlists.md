# Task: Wishlists (P3)

**Status:** not started  
**Track:** rails-store  
**Tutorial:** User Wishlists

## Goal

理解 join model、User↔Product 关系、DB constraint vs validation，以及 Turbo 在交互中的角色。

## Checklist

### Wishlist

- [ ] 创建 Wishlist 相关模型
- [ ] User ↔ Wishlist
- [ ] Wishlist ↔ Product
- [ ] 设计数据库关系
- [ ] 添加 unique constraint
- [ ] 添加 wishlist 页面
- [ ] 添加/删除商品
- [ ] 处理用户权限

### 重点理解

- [ ] `has_many` / `belongs_to` / `has_many :through`
- [ ] Join model
- [ ] Database constraints vs Rails validations
- [ ] Routes 设计
- [ ] Turbo/Hotwire 在交互中的作用

## Output

- [ ] 登录用户可增删 wishlist 商品
- [ ] Notes: `notes/p3-wishlists.md`（含关系图）

## Data model sketch

```text
User
  |
Wishlist / Join Model
  |
Product
```

## Skip

- Reviews（留给 P4）

## Notes

<!-- Session notes -->
