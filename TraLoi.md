# Trả lời câu hỏi bổ sung — Buổi 7

## Câu 1. Cấu trúc điều hướng

BookDetail được đăng ký ở Stack vì đây là màn hình đi sâu vào một cuốn sách, không phải một mục chính của app. Nằm ở Stack, màn này phủ lên cả Tab Bar và tự có nút ← để quay lại. Nếu đặt vào Tab, BookDetail sẽ thành một tab riêng: Tab Bar vẫn hiện, không có nút quay lại tự động, và tab này còn mở được khi chưa có `bookId`.

## Câu 2. Redux

`state` trong reducer không phải state gốc mà là bản nháp do Immer tạo ra. `state.items.push(...)` chỉ sửa bản nháp, Immer ghi lại các thay đổi rồi tạo ra state mới bất biến nên state gốc không bị đụng tới. Nếu bỏ `useMemo` trong `useCart`, kết quả hiển thị không đổi vì tổng vẫn được tính đúng. Hậu quả là tổng bị tính lại ở mỗi lần render, kể cả khi `items` không đổi; `useMemo` chỉ để tối ưu, giỏ càng lớn càng tốn hơn.

## Câu 3. Promise.all

Nếu `getCategories()` lỗi còn `getBooks()` thành công, Home hiện `ErrorView` và không có sách nào, vì `Promise.all` bị reject ngay khi một promise lỗi. Để vẫn hiện được sách, có thể dùng `Promise.allSettled`, rồi kiểm tra `status` của từng kết quả và chỉ gán dữ liệu khi là `"fulfilled"`. Đánh đổi là code dài hơn, phải tự xử lý lỗi cho từng phần, và phải quyết định giao diện khi thiếu dữ liệu: ví dụ sách vẫn hiện nhưng không có chip danh mục.

## Câu 4. Debounce

Nếu đặt `setTimeout` thẳng trong `onChangeText`, mỗi phím gõ tạo một hẹn giờ riêng và không hẹn giờ nào bị huỷ, nên danh sách vẫn lọc nhiều lần, chỉ là chậm đi 400 ms. Hàm dọn dẹp `return () => clearTimeout(t)` huỷ hẹn giờ cũ mỗi khi `value` đổi, nên chỉ lần gõ cuối được áp dụng. Nó còn tránh lỗi hẹn giờ chạy sau khi component đã bị gỡ, gây cập nhật state vô ích, và tránh việc kết quả cũ ghi đè lên giá trị mới.

## Câu 5. Code bổ sung: sắp xếp theo giá

Thêm một state `sort`, rồi sửa `visibleBooks` trong `src/screens/HomeScreen.tsx`. Phải sắp xếp trên bản sao `[...filtered]` vì `sort()` sửa trực tiếp mảng gốc. Ngoài ra có thêm 2 nút "Giá tăng dần" và "Giá giảm dần"; bấm lần nữa để bỏ sắp xếp.

```tsx
const [sort, setSort] = useState<"asc" | "desc" | null>(null);

const visibleBooks = useMemo(() => {
  const filtered = filterBooks(books, { keyword: debouncedKeyword, categoryId, priceRange });
  if (sort === null) return filtered;
  return [...filtered].sort((a, b) => (sort === "asc" ? a.price - b.price : b.price - a.price));
}, [books, debouncedKeyword, categoryId, priceRange, sort]);
```

Ảnh chụp màn hình kết quả: (đính kèm ảnh)

## Thử thách BT7.C1. Vì sao đổi thư viện state thì giao diện không phải sửa?

Giao diện không gọi trực tiếp thư viện state. `BookDetailScreen` chỉ dùng `useWishlist()` và lấy ra `isFavorite`, `toggle`, không import `useSelector`, `useDispatch` hay `wishlistSlice`. Mọi phần dính tới Redux nằm trong `src/store/hooks.ts`, file này đóng vai trò adapter đứng giữa giao diện và thư viện state. Nếu đổi sang Zustand, chỉ cần viết lại phần store và bên trong `useWishlist()`, miễn là hook vẫn trả về đúng `ids`, `isFavorite`, `toggle`, còn các file giao diện giữ nguyên.
