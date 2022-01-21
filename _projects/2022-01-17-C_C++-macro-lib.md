---
title: 'C/C++ macro lib'
date: 2022-01-17
permalink: /posts/2022/01/C_C++-macro-lib/
excerpt: 'Use macro to get macro parameters and implement other operations in C/C++.'
tags:
  - C/C++
  - Macro
  - Macro parameters
---

## Brief

Sometimes, we need to use macro to complete some miracle task. In this lib, you can use those macro to operate macro parameters.

## Function list

| func | desp |
| ---- | ---- |
| GET_MACRO_PARAS_TOTAL_NUM(...)  | get count of macro paras. |
| GET_MACRO_PARA_N(N, ...)  | get macro para in order N. |
| GET_MACRO_PARAS_N_2_END(N, ...)  | get macro paras from order N to end. |
| GET_MACRO_PARAS_START_2_N(N, ...) | get macro paras from order 0 to order N-1. |
| GET_MACRO_PARAS_N_2_M(N, M, ...) | get macro paras from order N to order M-1. |
| GET_MACRO_PARAS_N_LEN(N, LEN, ...) | get macro paras from order N to order N+LEN-1. |
| GET_MACRO_PARAS_RAM(N, ...) | get N macro paras in ramdom order. |
| MACRO_PARAS_ENUM_OPT(opt, ...) | provide 'opt' as a callback-func to every macro para. |
| MACRO_PARAS_ENUM_OPT_WITHOUT_LAST(opt, ...) | provide opt as a callback-func to every macro para except the last one. |
| MACRO_PARAS_NOTDEAL_0PARAS(opt, ...) | don't deal when macro paras is none. |

## Repository

Git repository:  
[github.com/novumdun/macro_func_collection](https://github.com/novumdun/macro_func_collection.git)  


