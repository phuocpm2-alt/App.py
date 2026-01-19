import streamlit as st
import pandas as pd
from openpyxl import load_workbook
import io

# Cấu hình trang web
st.set_page_config(page_title="Hệ thống Chấm điểm Thi đua", layout="centered")

st.title("🏆 Hệ thống Chấm điểm Thi đua")
st.markdown("Tải file dữ liệu thô và nhận lại file kết quả theo định dạng chuẩn.")

# --- PHẦN 1: TẢI FILE ---
col1, col2 = st.columns(2)
with col1:
    uploaded_file = st.file_uploader("1. Tải file Dữ liệu thô (Excel)", type=["xlsx"])
with col2:
    template_file = st.file_uploader("2. Tải file Template (Format)", type=["xlsx"])

# --- PHẦN 2: XỬ LÝ DỮ LIỆU ---
if uploaded_file and template_file:
    st.info("Đã nhận đủ file. Vui lòng kiểm tra cấu hình bên dưới.")
    
    # Giả định các thông số (Bạn có thể sửa lại cho khớp với file của mình)
    start_row = st.number_input("Dữ liệu trong Template bắt đầu từ dòng mấy?", value=5)
    
    if st.button("🚀 Bắt đầu Chấm điểm & Xuất File"):
        try:
            # Đọc dữ liệu thô
            df = pd.read_excel(uploaded_file)
            
            # KIỂM TRA VÀ TÍNH TOÁN (Đây là nơi bạn sửa logic)
            # Giả sử file có cột 'Tên', 'Lỗi', 'Thưởng'
            # Công thức: Điểm = 100 - (Lỗi * 5) + (Thưởng * 2)
            if 'Lỗi' in df.columns and 'Thưởng' in df.columns:
                df['Tổng Điểm'] = 100 - (df['Lỗi'] * 5) + (df['Thưởng'] * 2)
            else:
                # Nếu không tìm thấy cột, tạo cột giả định để không bị lỗi code
                st.warning("Không tìm thấy cột 'Lỗi' hoặc 'Thưởng', hệ thống sẽ lấy điểm mặc định 100.")
                df['Tổng Điểm'] = 100

            # Ghi vào Template
            template_bytes = template_file.read()
            wb = load_workbook(io.BytesIO(template_bytes))
            ws = wb.active
            
            # Lặp qua DataFrame và ghi vào file Excel
            # Giả sử: Cột B (2) ghi Tên, Cột C (3) ghi Tổng Điểm
            for i, row in df.iterrows():
                current_row = start_row + i
                ws.cell(row=current_row, column=2).value = row.get('Tên', 'N/A')
                ws.cell(row=current_row, column=3).value = row.get('Tổng Điểm', 0)
            
            # Xuất file ra bộ nhớ
            output = io.BytesIO()
            wb.save(output)
            processed_data = output.getvalue()
            
            st.success("✅ Xử lý thành công!")
            
            # Nút tải file
            st.download_button(
                label="📥 Tải file Kết quả (Excel)",
                data=processed_data,
                file_name="Ket_qua_thi_dua_cuoi_cung.xlsx",
                mime="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
            )
            
        except Exception as e:
            st.error(f"Có lỗi xảy ra: {e}")

else:
    st.warning("Vui lòng tải lên cả 2 file để bắt đầu.")

# Hướng dẫn nhỏ
with st.expander("Hướng dẫn sử dụng"):
    st.write("""
    1. **File dữ liệu thô:** Phải có các cột tiêu đề như 'Tên', 'Lỗi', 'Thưởng'.
    2. **File Template:** Là file trắng đã kẻ bảng, có logo... ứng dụng sẽ điền đè dữ liệu vào.
    3. **Dòng bắt đầu:** Nếu file của bạn có tiêu đề ở dòng 1-4, hãy nhập số 5.
    """)
# App.py
