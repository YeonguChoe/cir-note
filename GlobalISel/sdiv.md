# Optimize `sdiv` CodeGen

```llvm
define <32 x i8> @divide_by_two(<32 x i8> %dividend) {
entry:
  %dividend.ptr = alloca <32 x i8>, align 32
  %divisor.ptr = alloca <32 x i8>, align 32
  store <32 x i8> %dividend, ptr %dividend.ptr, align 32
  store <32 x i8> <i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2, i8 2>, ptr %divisor.ptr, align 32
  %dividend.val = load <32 x i8>, ptr %dividend.ptr, align 32
  %divisor.val = load <32 x i8>, ptr %divisor.ptr, align 32
  %quotient = sdiv <32 x i8> %dividend.val, %divisor.val
  ret <32 x i8> %quotient
}
```
