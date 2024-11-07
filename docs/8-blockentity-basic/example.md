# 8.6 示例

我们曾经在4.2演示过通过方块状态来实现水箱的水位，但是每个水位的方块状态就需要一个模型，那么如果我们想要更加自由的操作，就可以使用方块实体

## 水位数据

根据前一章学到的，首先在方块实体中添加一个字段waterLevel，然后使数据持久化，也就是load和saveAdditional方法

``` java
    public void load(@NotNull CompoundTag pTag) {
        super.load(pTag);
        this.waterLevel = getPersistentData().getInt("water_level");
    }

    protected void saveAdditional(@NotNull CompoundTag pTag) {
        getPersistentData().putInt("water_level", this.waterLevel);
        super.saveAdditional(pTag);
    }
```

## 交互

你直接右击交互的是BaseEntityBlock基础实体方块，这时候就需要覆写use方法来实现与方块实体的交互

``` java
    @SuppressWarnings("deprecation")
    @Override
    public @NotNull InteractionResult use(BlockState blockState, Level level, BlockPos blockPos, Player player, InteractionHand interactionHand, BlockHitResult result) {
        if (!level.isClientSide) {
            BlockEntity blockEntity = level.getBlockEntity(blockPos);
            return InteractionResult.SUCCESS;
        }
        return InteractionResult.CONSUME;
    }
```

这时候，你就拿到了服务端上的方块实体对象。下面就是在服务端上对这个实例进行操作

``` java
            if (blockEntity instanceof TestTE TestTE) {
                ItemStack mainHandItem = player.getMainHandItem().copy();
                if (mainHandItem.is(Items.WATER_BUCKET)) {
                    testTE.waterLevel++;
                    player.setItemInHand(interactionHand, Items.BUCKET.getDefaultInstance());
                    level.sendBlockUpdated(blockPos, blockState, blockState, Block.UPDATE_CLIENTS);
                }
            }
```

这里首先检测手上物品是否是水桶，如果是，就使得方块实体中的waterLevel自增，并把玩家手上的物品设置为桶。最后再告诉mc，把服务端上方块实体的数据同步到客户端上。

下面就是额外内容，也就是渲染

## 渲染

首先，我们得拿到原版水的精灵图

``` java
    private static Optional<TextureAtlasSprite> getStillFluidSprite() {
        TextureAtlasSprite sprite = Minecraft.getInstance().getTextureAtlas(InventoryMenu.BLOCK_ATLAS).apply(WATER_STILL);
        return Optional.of(sprite).filter(s -> s.atlasLocation() != MissingTextureAtlasSprite.getLocation());
    }
```

然后在render方法里，通过写顶点的方式，渲染出一个水平面

``` java
        if (crucibleTE.waterLevel > 0) {
            getStillFluidSprite().ifPresent(fluidSprite -> {
                poseStack.pushPose();
                int waterLevel = crucibleTE.waterLevel;
                // long hue = (System.currentTimeMillis() / 10) % 360;
                // int[] rgb = HSLtoRGB(hue, 0.5f, 0.5f);
                int[] rgb = new int[]{255,255,255}
                float height = waterLevel / 16.0F + 0.15F;
                Matrix4f matrix4f = poseStack.last().pose();
                RenderType renderType = RenderType.translucent();
                VertexConsumer vertexConsumer = pBuffer.getBuffer(renderType);
                float r = rgb[0];
                float g = rgb[1];
                float b = rgb[2];
                float alpha = 1.0f;
                vertexConsumer.vertex(matrix4f, 0f, height, 0f)
                        .color(r, g, b, alpha)
                        .uv(fluidSprite.getU0(), fluidSprite.getV0())
                        .overlayCoords(OverlayTexture.NO_OVERLAY)
                        .uv2(pPackedLight).normal(0.0F, 1.0F, 0.0F).endVertex();

                vertexConsumer.vertex(matrix4f, 0f, height, 1f)
                        .color(r, g, b, alpha)
                        .uv(fluidSprite.getU0(), fluidSprite.getV1())
                        .overlayCoords(OverlayTexture.NO_OVERLAY)
                        .uv2(pPackedLight).normal(0.0F, 1.0F, 0.0F).endVertex();

                vertexConsumer.vertex(matrix4f, 1f, height, 1f)
                        .color(r, g, b, alpha)
                        .uv(fluidSprite.getU1(), fluidSprite.getV1())
                        .overlayCoords(OverlayTexture.NO_OVERLAY)
                        .uv2(pPackedLight).normal(0.0F, 1.0F, 0.0F).endVertex();

                vertexConsumer.vertex(matrix4f, 1f, height, 0f)
                        .color(r, g, b, alpha)
                        .uv(fluidSprite.getU1(), fluidSprite.getV0())
                        .overlayCoords(OverlayTexture.NO_OVERLAY)
                        .uv2(pPackedLight).normal(0.0F, 1.0F, 0.0F).endVertex();
                poseStack.popPose();
            });
        }
```

这段代码中，我们可以看到这个部分重复了四次，也就是这个面的四个顶点，下面我们来逐步剖析
``` java
                vertexConsumer.vertex(matrix4f, 1f, height, 0f)
                        .color(r, g, b, alpha)
                        .uv(fluidSprite.getU1(), fluidSprite.getV0())
                        .overlayCoords(OverlayTexture.NO_OVERLAY)
                        .uv2(pPackedLight).normal(0.0F, 1.0F, 0.0F).endVertex();
```
.vertex开始写入的顶点数据，matrix4f是当前的位姿，后面三个参数是顶点坐标的x,y,z

.color写入rgba数据，也就是颜色和透明度

.uv写入这个顶点在对应精灵图上的uv坐标

.overlayCoord详情可以看这篇教程 [OverlayTexture](https://zomb-676.github.io/CobaltDocs/#/render/overlayTexture)

.uv2写入光照数据

.normal写入法线数据

.endVertex封装顶点数据

那么我们分别写入四个坐标为(0,height,0) (0,height,1) (1,height,1) (1,height,0)的顶点，就在height高度，渲染出来一个白色的1*1的水面

那如果我们想要一个原版水的效果呢，就可以手动设置一下rgb数组中的数据了

## 私货

在上述代码中还有一个被注释掉的部分
``` java
    long hue = (System.currentTimeMillis() / 10) % 360;
    int[] rgb = HSLtoRGB(hue, 0.5f, 0.5f);
```
第一行是拿到系统时间然后取模360，来获得一个每3.6秒在0-360之间循环的值
下一步是把这个数字作为色相，现了 HSL 到 RGB 的转换，这样我们就有了一个rgb炫彩变色水面了

``` java
    public static int[] HSLtoRGB(float hue, float saturation, float lightness) {
        float c = (1 - Math.abs(2 * lightness - 1)) * saturation;
        float x = c * (1 - Math.abs((hue / 60) % 2 - 1));
        float m = lightness - c / 2;

        float r, g, b;
        if (hue < 60) {
            r = c;
            g = x;
            b = 0;
        } else if (hue < 120) {
            r = x;
            g = c;
            b = 0;
        } else if (hue < 180) {
            r = 0;
            g = c;
            b = x;
        } else if (hue < 240) {
            r = 0;
            g = x;
            b = c;
        } else if (hue < 300) {
            r = x;
            g = 0;
            b = c;
        } else {
            r = c;
            g = 0;
            b = x;
        }

        int red = (int) ((r + m) * 255);
        int green = (int) ((g + m) * 255);
        int blue = (int) ((b + m) * 255);

        return new int[]{red, green, blue};
    }
```