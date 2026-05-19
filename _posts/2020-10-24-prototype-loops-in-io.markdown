---
layout:     post
date:       2020-10-24 18:44:42 -0400 EDT
title:      "Prototype loops in Io"
categories: Software
---

The default nature of Io is that, if you follow the prototypes up from the Lobby, you come back to the Lobby.

I didn't realize that, before today.

Let me summarize what the REPL shows below:

- _Lobby_ is `Object_0x7fb6e7c1e330`
- _Protos_ is `Object_0x7fb6e7c1e240`
- _Core_ is `Object_0x7fb6e7c1e2b0`
- _Object_ is `Object_0x7fb6e7c05e30`

What's curious, and maybe it's obvious to someone else, is why you don't see Lobby as a slot in Object.  I didn't omit any part of its slotSummary, and Lobby just isn't there.  But you can `getSlot` it, anyhow.  And it responds to `Object Lobby`.  I just don't get it.

	Io> Lobby slotSummary
	==>  Object_0x7fb6e7c1e330:
	  Lobby            = Object_0x7fb6e7c1e330
	  Protos           = Object_0x7fb6e7c1e240
	  _                = nil
	  exit             = method(...)
	  forward          = method(...)
	  set_             = method(...)

	Io> Lobby proto slotSummary
	==>  Object_0x7fb6e7c1e240:
	  Addons           = Object_0x7fb6e7c1e800
	  Core             = Object_0x7fb6e7c1e2b0

	Io> Lobby proto proto slotSummary
	==>  Object_0x7fb6e7c1e2b0:
	...
	  Number           = 0
	  Object           = Object_0x7fb6e7c05e30
	  OperatorTable    = OperatorTable_0x7fb6e7e1edb0
	...
	  false            = false
	  nil              = nil
	  tildeExpandsTo   = method(...)
	  true             = true
	  vector           = method(...)

	Io> Lobby proto proto proto slotSummary
	==>  Object_0x7fb6e7c05e30:
	                   = Object_()
	  !=               = Object_!=()
	  -                = Object_-()
	  ..               = method(arg, ...)
	  <                = Object_<()
	  <=               = Object_<=()
	  ==               = Object_==()
	  >                = Object_>()
	  >=               = Object_>=()
	  ?                = method(...)
	  @                = method(...)
	  @@               = method(...)
	  actorProcessQueue = method(...)
	  actorRun         = method(...)
	  addTrait         = method(obj, ...)
	  ancestorWithSlot = Object_ancestorWithSlot()
	  ancestors        = method(a, ...)
	  and              = method(v, ...)
	  appendProto      = Object_appendProto()
	  apropos          = method(keyword, ...)
	  argIsActivationRecord = Object_argIsActivationRecord()
	  argIsCall        = Object_argIsCall()
	  asBoolean        = Object_asBoolean()
	  asSimpleString   = method(...)
	  asString         = method(keyword, ...)
	  asyncSend        = method(...)
	  become           = Object_become()
	  block            = Object_block()
	  break            = Object_break()
	  clone            = Object_clone()
	  cloneWithoutInit = Object_cloneWithoutInit()
	  compare          = Object_compare()
	  contextWithSlot  = Object_contextWithSlot()
	  continue         = Object_continue()
	  coroDo           = method(...)
	  coroDoLater      = method(...)
	  coroFor          = method(...)
	  coroWith         = method(...)
	  currentCoro      = method(...)
	  deprecatedWarning = method(newName, ...)
	  do               = Object_do()
	  doFile           = Object_doFile()
	  doMessage        = Object_doMessage()
	  doRelativeFile   = method(path, ...)
	  doString         = Object_doString()
	  evalArg          = Object_evalArg()
	  evalArgAndReturnNil = Object_evalArgAndReturnNil()
	  evalArgAndReturnSelf = Object_evalArgAndReturnSelf()
	  for              = Object_for()
	  foreachSlot      = method(...)
	  futureSend       = method(...)
	  getLocalSlot     = Object_getLocalSlot()
	  getSlot          = Object_getSlot()
	  handleActorException = method(e, ...)
	  hasDirtySlot     = Object_hasDirtySlot()
	  hasLocalSlot     = Object_hasLocalSlot()
	  hasProto         = Object_hasProto()
	  hasSlot          = method(n, ...)
	  if               = Object_if()
	  ifError          = method(...)
	  ifNil            = Object_thisContext()
	  ifNilEval        = Object_thisContext()
	  ifNonNil         = Object_evalArgAndReturnSelf()
	  ifNonNilEval     = Object_evalArg()
	  in               = method(aList, ...)
	  init             = Object_init()
	  inlineMethod     = method(...)
	  isActivatable    = Object_isActivatable()
	  isError          = false
	  isIdenticalTo    = Object_isIdenticalTo()
	  isKindOf         = method(anObject, ...)
	  isLaunchScript   = method(...)
	  isNil            = false
	  isTrue           = true
	  justSerialized   = method(stream, ...)
	  launchFile       = method(path, args, ...)
	  lazySlot         = method(...)
	  lexicalDo        = Object_lexicalDo()
	  list             = method(...)
	  loop             = Object_loop()
	  markClean        = Object_markClean()
	  memorySize       = Object_memorySize()
	  message          = Object_message()
	  method           = Object_method()
	  newSlot          = method(name, value, doc, ...)
	  not              = nil
	  or               = true
	  ownsSlots        = Object_ownsSlots()
	  pause            = method(...)
	  perform          = Object_perform()
	  performWithArgList = Object_performWithArgList()
	  prependProto     = Object_prependProto()
	  print            = method(...)
	  println          = method(...)
	  proto            = Object_proto()
	  protos           = Object_protos()
	  raiseIfError     = method(...)
	  relativeDoFile   = method(path, ...)
	  removeAllProtos  = Object_removeAllProtos()
	  removeAllSlots   = Object_removeAllSlots()
	  removeProto      = Object_removeProto()
	  removeSlot       = Object_removeSlot()
	  resend           = method(...)
	  return           = Object_return()
	  returnIfError    = method(...)
	  returnIfNonNil   = Object_returnIfNonNil()
	  serialized       = method(stream, ...)
	  serializedSlots  = method(stream, ...)
	  serializedSlotsWithNames = method(names, stream, ...)
	  setIsActivatable = Object_setIsActivatable()
	  setProto         = Object_setProto()
	  setProtos        = Object_setProtos()
	  setSlot          = Object_setSlot()
	  setSlotWithType  = Object_setSlotWithType()
	  shallowCopy      = Object_shallowCopy()
	  slotDescriptionMap = method(...)
	  slotNames        = Object_slotNames()
	  slotSummary      = method(keyword, ...)
	  slotValues       = Object_slotValues()
	  stopStatus       = Object_stopStatus()
	  super            = method(...)
	  switch           = method(...)
	  thisContext      = Object_thisContext()
	  thisLocalContext = Object_thisLocalContext()
	  thisMessage      = Object_thisMessage()
	  try              = method(...)
	  type             = Object_type()
	  uniqueHexId      = method(...)
	  uniqueId         = Object_uniqueId()
	  updateSlot       = Object_updateSlot()
	  wait             = method(s, ...)
	  while            = Object_while()
	  write            = Object_write()
	  writeln          = Object_writeln()
	  yield            = method(...)

	Io> Lobby proto proto proto proto slotSummary
	==>  Object_0x7fb6e7c1e330:
	  Lobby            = Object_0x7fb6e7c1e330
	  Protos           = Object_0x7fb6e7c1e240
	  _                = "..."

	Io> 

