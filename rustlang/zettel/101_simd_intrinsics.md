# SIMD Intrinsics in Rust

## 핵심 개념
Single Instruction Multiple Data - 벡터 연산으로 병렬 처리

## 포터블 SIMD (std::simd)
```rust
#![feature(portable_simd)]
use std::simd::{f32x4, f32x8, SimdFloat};

fn dot_product_simd(a: &[f32], b: &[f32]) -> f32 {
    assert_eq!(a.len(), b.len());
    let chunks = a.len() / 4;
    
    let mut sum = f32x4::splat(0.0);
    
    for i in 0..chunks {
        let a_vec = f32x4::from_slice(&a[i*4..]);
        let b_vec = f32x4::from_slice(&b[i*4..]);
        sum += a_vec * b_vec;
    }
    
    // 나머지 처리
    let mut result = sum.reduce_sum();
    for i in chunks*4..a.len() {
        result += a[i] * b[i];
    }
    
    result
}
```

## 플랫폼별 인트린식
```rust
#[cfg(target_arch = "x86_64")]
use std::arch::x86_64::*;

#[target_feature(enable = "avx2")]
unsafe fn add_vectors_avx2(a: &[f32], b: &[f32], result: &mut [f32]) {
    let len = a.len();
    let simd_len = len - (len % 8);
    
    for i in (0..simd_len).step_by(8) {
        let a_vec = _mm256_loadu_ps(a.as_ptr().add(i));
        let b_vec = _mm256_loadu_ps(b.as_ptr().add(i));
        let sum = _mm256_add_ps(a_vec, b_vec);
        _mm256_storeu_ps(result.as_mut_ptr().add(i), sum);
    }
    
    // 스칼라 폴백
    for i in simd_len..len {
        result[i] = a[i] + b[i];
    }
}
```

## 런타임 기능 감지
```rust
fn process_data(data: &mut [f32]) {
    #[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
    {
        if is_x86_feature_detected!("avx2") {
            unsafe { process_avx2(data) }
        } else if is_x86_feature_detected!("sse4.1") {
            unsafe { process_sse41(data) }
        } else {
            process_scalar(data)
        }
    }
    
    #[cfg(not(any(target_arch = "x86", target_arch = "x86_64")))]
    process_scalar(data)
}
```

## 벡터화 가능한 패턴
```rust
// 자동 벡터화 친화적
fn sum_arrays(a: &[f32], b: &[f32], out: &mut [f32]) {
    // 컴파일러가 SIMD로 변환 가능
    for i in 0..a.len() {
        out[i] = a[i] + b[i];
    }
}

// 수동 벡터화
use packed_simd_2::{f32x8, Simd};

fn sum_arrays_simd(a: &[f32], b: &[f32], out: &mut [f32]) {
    let chunks = a.len() / 8;
    
    for i in 0..chunks {
        let idx = i * 8;
        let a_vec = f32x8::from_slice_unaligned(&a[idx..]);
        let b_vec = f32x8::from_slice_unaligned(&b[idx..]);
        (a_vec + b_vec).write_to_slice_unaligned(&mut out[idx..]);
    }
    
    // 나머지
    for i in chunks*8..a.len() {
        out[i] = a[i] + b[i];
    }
}
```

## 마스크 연산
```rust
use std::simd::{mask32x4, f32x4};

fn threshold_filter(data: &mut [f32], threshold: f32) {
    let threshold_vec = f32x4::splat(threshold);
    
    for chunk in data.chunks_exact_mut(4) {
        let vec = f32x4::from_slice(chunk);
        let mask = vec.simd_gt(threshold_vec);
        let result = mask.select(vec, f32x4::splat(0.0));
        result.copy_to_slice(chunk);
    }
}
```

## 셔플과 퍼뮤테이션
```rust
#[cfg(target_arch = "x86_64")]
unsafe fn transpose_4x4(matrix: &mut [[f32; 4]; 4]) {
    use std::arch::x86_64::*;
    
    let row0 = _mm_loadu_ps(matrix[0].as_ptr());
    let row1 = _mm_loadu_ps(matrix[1].as_ptr());
    let row2 = _mm_loadu_ps(matrix[2].as_ptr());
    let row3 = _mm_loadu_ps(matrix[3].as_ptr());
    
    let tmp0 = _mm_unpacklo_ps(row0, row1);
    let tmp1 = _mm_unpackhi_ps(row0, row1);
    let tmp2 = _mm_unpacklo_ps(row2, row3);
    let tmp3 = _mm_unpackhi_ps(row2, row3);
    
    let col0 = _mm_movelh_ps(tmp0, tmp2);
    let col1 = _mm_movehl_ps(tmp2, tmp0);
    let col2 = _mm_movelh_ps(tmp1, tmp3);
    let col3 = _mm_movehl_ps(tmp3, tmp1);
    
    _mm_storeu_ps(matrix[0].as_mut_ptr(), col0);
    _mm_storeu_ps(matrix[1].as_mut_ptr(), col1);
    _mm_storeu_ps(matrix[2].as_mut_ptr(), col2);
    _mm_storeu_ps(matrix[3].as_mut_ptr(), col3);
}
```

## 문자열 처리 SIMD
```rust
fn find_byte_simd(haystack: &[u8], needle: u8) -> Option<usize> {
    use std::simd::{u8x32, SimdPartialEq};
    
    let needle_vec = u8x32::splat(needle);
    
    for (i, chunk) in haystack.chunks_exact(32).enumerate() {
        let vec = u8x32::from_slice(chunk);
        let matches = vec.simd_eq(needle_vec);
        if matches.any() {
            // 첫 번째 매치 위치 찾기
            for (j, &byte) in chunk.iter().enumerate() {
                if byte == needle {
                    return Some(i * 32 + j);
                }
            }
        }
    }
    
    // 나머지 검색
    for (i, &byte) in haystack[haystack.len() - haystack.len() % 32..].iter().enumerate() {
        if byte == needle {
            return Some(haystack.len() - haystack.len() % 32 + i);
        }
    }
    
    None
}
```

## 수평 연산
```rust
use std::simd::{f32x8, SimdFloat};

fn horizontal_sum(values: &[f32]) -> f32 {
    let mut sum = f32x8::splat(0.0);
    
    for chunk in values.chunks_exact(8) {
        sum += f32x8::from_slice(chunk);
    }
    
    let mut result = sum.reduce_sum();
    
    // 나머지
    for &val in &values[values.len() - values.len() % 8..] {
        result += val;
    }
    
    result
}
```

## Rust 특성 활용
- **안전한 추상화**: 포터블 SIMD API
- **조건부 컴파일**: 플랫폼별 최적화
- **제로 비용**: 직접 인트린식 매핑
- **타입 안전**: SIMD 타입 체크

## 성능 고려사항
- 데이터 정렬 중요
- 브랜치 제거
- 캐시 친화적 접근
- 벡터 레지스터 활용도
- 폴백 구현 필수

## 관련 개념
- [[100_benchmarking]] - 성능 측정
- [[102_cache_optimization]] - 캐시 최적화
- [[103_memory_alignment]] - 메모리 정렬