<!-- master#657b856 --->

# 处理循环

默认情况下，当 Salsa 在计算图中检测到循环时，Salsa 会将 [`salsa::Cycle`] 的值作为 panic 的值。

描述循环的 [`salsa::Cycle`] 结构对发现哪里出了问题很有用。

[`salsa::Cycle`]: https://github.com/salsa-rs/salsa/blob/0f9971ad94d5d137f1192fde2b02ccf1d2aca28c/src/lib.rs#L654-L672

