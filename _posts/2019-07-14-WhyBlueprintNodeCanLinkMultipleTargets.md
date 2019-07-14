---
layout: post
title: 为什么某些BlueprintNode的Target可以连接多个对象？
date: 2019-07-14
category: notes
tags: [unreal4,c++]
---

### Description
<center>允许多个Link</center>
<img src="/img/in-post/MultipleLinksAllowed.png" alt="AllowMutipleLinks" width="50%" />
<center>不允许多个Link</center>
<img src="/img/in-post/MultipleLinksUnallowed.gif" alt="MultipleLinksUnallowed" width="50%" />

### Cause
查看两个UFUNCTION的specifier
``` C++
// Allow multple links to Target
UFUNCTION(BlueprintCallable, Category=Character)
       virtual void Jump();

// Unallow
UFUNCTION(BlueprintCallable, meta=(ScriptNoExport, BlueprintInternalUseOnly = "true", DefaultToSelf="ComponentTemplateContext", InternalUseParam="ComponentTemplateContext"))
       class UActorComponent* AddComponent(FName TemplateName, bool bManualAttachment, const FTransform& RelativeTransform, const UObject* ComponentTemplateContext);
```
看上去没有什么和Pin直接有关系的，只不过蓝图中是`AddCameraComponent`，而源代码则指向了一个通用的`AddComponent`，也就是说，蓝图中的这个函数结点是根据通用函数生成出来的，并没有实际的函数对应。

于是顺着结点生成的相关代码找去，通过Debugger捕捉Link和Unlink的行为，终于发现和下面这个函数有关，大家重点关注bMultipleSelfException的赋值：
``` C++
const FPinConnectionResponse UEdGraphSchema_K2::DetermineConnectionResponseOfCompatibleTypedPins(const UEdGraphPin* PinA, const UEdGraphPin* PinB, const UEdGraphPin* InputPin, const UEdGraphPin* OutputPin) const
{
	// Now check to see if there are already connections and this is an 'exclusive' connection
	const bool bBreakExistingDueToExecOutput = IsExecPin(*OutputPin) && (OutputPin->LinkedTo.Num() > 0);
	const bool bBreakExistingDueToDataInput = !IsExecPin(*InputPin) && (InputPin->LinkedTo.Num() > 0);

	bool bMultipleSelfException = false;
	const UK2Node* OwningNode = Cast<UK2Node>(InputPin->GetOwningNode());
	if (bBreakExistingDueToDataInput && 
		IsSelfPin(*InputPin) && 
		OwningNode &&
		OwningNode->AllowMultipleSelfs(false) &&
		!InputPin->PinType.IsContainer() &&
		!OutputPin->PinType.IsContainer() )
	{
		//check if the node wont be expanded as foreach call, if there is a link to an array
		bool bAnyContainerInput = false;
		for(int InputLinkIndex = 0; InputLinkIndex < InputPin->LinkedTo.Num(); InputLinkIndex++)
		{
			if(const UEdGraphPin* Pin = InputPin->LinkedTo[InputLinkIndex])
			{
				if(Pin->PinType.IsContainer())
				{
					bAnyContainerInput = true;
					break;
				}
			}
		}
		bMultipleSelfException = !bAnyContainerInput;
	}

	if (bBreakExistingDueToExecOutput)
	{
		const ECanCreateConnectionResponse ReplyBreakOutputs = (PinA == OutputPin) ? CONNECT_RESPONSE_BREAK_OTHERS_A : CONNECT_RESPONSE_BREAK_OTHERS_B;
		return FPinConnectionResponse(ReplyBreakOutputs, TEXT("Replace existing output connections"));
	}
	else if (bBreakExistingDueToDataInput && !bMultipleSelfException)
	{
		const ECanCreateConnectionResponse ReplyBreakInputs = (PinA == InputPin) ? CONNECT_RESPONSE_BREAK_OTHERS_A : CONNECT_RESPONSE_BREAK_OTHERS_B;
		return FPinConnectionResponse(ReplyBreakInputs, TEXT("Replace existing input connections"));
	}
	else
	{
		return FPinConnectionResponse(CONNECT_RESPONSE_MAKE, TEXT(""));
	}
}
```
看来只有Target（也就是CPP函数的this）的Pin可以连接，并且使用的函数结点其`AllowMultipleSelfs()`也要返回True，至于其他条件代码写的很明白就不说了。

不同的结点类型，允许连接多个对象的条件不同，譬如`UK2Node_CallFunction`这个类型就会以函数是否有返回值作为标准，如下：
``` C++
bool UK2Node_CallFunction::AllowMultipleSelfs(bool bInputAsArray) const
{
	if (UFunction* Function = GetTargetFunction())
	{
		return CanFunctionSupportMultipleTargets(Function);
	}

	return Super::AllowMultipleSelfs(bInputAsArray);
}

bool UK2Node_CallFunction::CanFunctionSupportMultipleTargets(UFunction const* Function)
{
	bool const bIsImpure = !Function->HasAnyFunctionFlags(FUNC_BlueprintPure);
	bool const bIsLatent = Function->HasMetaData(FBlueprintMetadata::MD_Latent);
	bool const bHasReturnParam = (Function->GetReturnProperty() != nullptr);

	return !bHasReturnParam && bIsImpure && !bIsLatent;
}
```