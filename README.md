# CIR practice

## Command
- Converting C/C++ to CIR

```bash
/tmp/install-llvm/bin/clang -fclangir -emit-cir -clangir-disable-passes  main.c -o main.cir
```

- Converting CIR to LLVM IR

```bash
/tmp/install-llvm/bin/cir-translate --cir-to-llvmir main.cir -o main.ll
```

- Converting C/C++ to LLVM IR

```bash
/tmp/install-llvm/bin/clang -fclangir -S -emit-llvm main.c
```

```cpp
    CodeGenFunction::CGFPOptionsRAII FPOptsRAII(*this, E);
    Value *V = EmitScalarExpr(E->getArg(5));

    Value *NanLiteral = EmitScalarExpr(E->getArg(0));
    Value *InfLiteral = EmitScalarExpr(E->getArg(1));
    Value *NormalLiteral = EmitScalarExpr(E->getArg(2));
    Value *SubnormalLiteral = EmitScalarExpr(E->getArg(3));
    Value *ZeroLiteral = EmitScalarExpr(E->getArg(4));

    Value *IsNan = Builder.createIsFPClass(V, 0b0000000011);
    Value *IsInf = Builder.createIsFPClass(V, 0b1000000100);
    Value *IsNormal = Builder.createIsFPClass(V, 0b0100001000);
    Value *IsSubnormal = Builder.createIsFPClass(V, 0b0010010000);

    BasicBlock *Entry = Builder.GetInsertBlock();

    BasicBlock *End = createBasicBlock("fpclassify_end", CurFn);
    Builder.SetInsertPoint(End);
    PHINode *Result = Builder.CreatePHI(ConvertType(E->getArg(0)->getType()), 5,
                                        "fpclassify_result");

    // Check if V is NaN
    Builder.SetInsertPoint(Entry);
    BasicBlock *NotNan = createBasicBlock("fpclassify_not_nan", CurFn);
    Builder.CreateCondBr(IsNan, End, NotNan);
    Result->addIncoming(NanLiteral, Entry);

    // Check if V is infinity
    Builder.SetInsertPoint(NotNan);
    BasicBlock *NotInf = createBasicBlock("fpclassify_not_inf", CurFn);
    Builder.CreateCondBr(IsInf, End, NotInf);
    Result->addIncoming(InfLiteral, NotNan);

    // Check if V is normal
    Builder.SetInsertPoint(NotInf);
    BasicBlock *NotNormal = createBasicBlock("fpclassify_not_normal", CurFn);
    Builder.CreateCondBr(IsNormal, End, NotNormal);
    Result->addIncoming(NormalLiteral, NotInf);

    // Check if V is subnormal
    Builder.SetInsertPoint(NotNormal);
    BasicBlock *NotSubnormal =
        createBasicBlock("fpclassify_not_subnormal", CurFn);
    Builder.CreateCondBr(IsSubnormal, End, NotSubnormal);
    Result->addIncoming(SubnormalLiteral, NotNormal);

    // If V is not one of the above, it is zero
    Builder.SetInsertPoint(NotSubnormal);
    Builder.CreateBr(End);
    Result->addIncoming(ZeroLiteral, NotSubnormal);

    Builder.SetInsertPoint(End);
    return RValue::get(Result);
```
