# 9.2 基础

## 交互
如果你直接继承自Entity类，你会发现实体空有碰撞箱却无法交互，你需要覆写`pickable`方法，这个方法默认返回false，即交互时会略过这个实体

``` java
    /**
     * Returns {@code true} if other Entities should be prevented from moving through this Entity.
     */
    public boolean isPickable() {
        return !this.isRemoved();
    }
```

下面的交互逻辑在`interact`方法中

``` java
    @Override
    public @NotNull InteractionResult interact(@NotNull Player pPlayer, @NotNull InteractionHand pHand) {
        Level level = pPlayer.level();
        if (!level.isClientSide) {

        }
        return InteractionResult.PASS;
    }
```

## 骑乘

如果想实现骑乘，需要继承自Mob类，然后覆写以下方法来实现玩家控制被骑乘实体的行为逻辑

``` java
    @Nullable
    public LivingEntity getControllingPassenger() {
        Entity entity = this.getFirstPassenger();
        if (entity instanceof Player player) {
            return player;
        }
        return null;
    }

    protected @NotNull Vec3 getRiddenInput(Player player, @NotNull Vec3 vec3) {
        float f = player.xxa * 0.5F;
        float f1 = player.zza;
        if (f1 <= 0.0F) {
            f1 *= 0.25F;
        }
        return new Vec3(f, f1 * Math.sin(Math.toRadians( - player.getXRot())), f1);
    }

    protected void tickRidden(@NotNull Player player, @NotNull Vec3 vec3) {
        super.tickRidden(player, vec3);
        this.setRot(player.getYRot(), player.getXRot() * 0.5F);
        this.yRotO = this.yBodyRot = this.yHeadRot = this.getYRot();
    }

    protected float getRiddenSpeed(@NotNull Player player) {
        return (float)(this.getAttributeValue(Attributes.MOVEMENT_SPEED));
    }
```

当然很重要的一点就是开始骑乘，可以在玩家右击实体的事件中实现

``` java
    public @NotNull InteractionResult mobInteract(@NotNull Player player, @NotNull InteractionHand hand) {
        if (!this.level().isClientSide) {
            player.startRiding(this);
        }
        return InteractionResult.sidedSuccess(this.level().isClientSide);
    }

```
