from selenium import webdriver
from selenium.webdriver.firefox.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.firefox.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time

# Đường dẫn đến GeckoDriver (nên sử dụng một đường dẫn ổn định)
gecko_driver_path = r"C:\geckodriver\geckodriver.exe"  # Cập nhật lại đường dẫn này

# Thiết lập tùy chọn cho Firefox
options = Options()
# options.headless = True  # Bỏ dòng này để hiển thị trình duyệt

# Khởi tạo dịch vụ GeckoDriver
service = Service(executable_path=gecko_driver_path)

# Khởi tạo trình duyệt Firefox với GeckoDriver
driver = webdriver.Firefox(service=service, options=options)

try:
    # Truy cập trang đầu tiên
    driver.get("https://bds123.vn/nha-dat-ban-ho-chi-minh.html")
    
    while True:
        # Đợi trang tải đầy đủ
        WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.XPATH, '//*[@id="main"]/div[2]/section/ul[1]/li')))
        
        # Lấy danh sách các mục tin
        items = driver.find_elements(By.XPATH, '//*[@id="main"]/div[2]/section/ul[1]/li')

        print("Thu thập dữ liệu trên Trang hiện tại:")
        
        for index, item in enumerate(items, start=1):
            try:
                # Cuộn đến từng tin và chờ 2 giây trước khi tiếp tục
                driver.execute_script("arguments[0].scrollIntoView();", item)
                time.sleep(2)  # Đợi 2 giây sau khi cuộn để tránh quá nhanh
                
                # Thu thập thông tin
                gia = item.find_element(By.XPATH, './a/aside/div[1]/span[1]').text
                dien_tich = item.find_element(By.XPATH, './a/aside/div[1]/div/span[1]').text
                phong_ngu = item.find_element(By.XPATH, './a/aside/div[1]/div/span[2]').text
                nha_tam = item.find_element(By.XPATH, './a/aside/div[1]/div/span[3]').text
                dia_chi = item.find_element(By.XPATH, './a/aside/div[2]/p/span').text

                # In thông tin ra console
                print(f"Tin thứ {index}:")
                print(f"Giá: {gia}")
                print(f"Diện tích: {dien_tich}")
                print(f"Phòng ngủ: {phong_ngu}")
                print(f"Nhà tắm: {nha_tam}")
                print(f"Địa chỉ: {dia_chi}")
                print("-" * 50)

            except Exception as e:
                print(f"Lỗi khi thu thập tin thứ {index}: {e}")
        
        # Tìm số trang tiếp theo và bấm vào
        try:
            # Xác định trang hiện tại
            current_page = driver.find_element(By.XPATH, '//li[@class="page-item active"]/span').text

            # Tìm số trang tiếp theo
            next_page = driver.find_element(By.XPATH, f'//li[@class="page-item"]/a[@class="page-link" and text()="{int(current_page) + 1}"]')
            
            print(f"Nhấn vào số trang {int(current_page) + 1} để chuyển sang trang tiếp theo.")
            next_page.click()
            
            # Đợi trang mới tải xong trước khi tiếp tục
            WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.XPATH, '//*[@id="main"]/div[2]/section/ul[1]/li')))
            
        except Exception:
            print("Không tìm thấy trang tiếp theo. Quá trình thu thập đã hoàn tất.")
            break

except Exception as e:
    print(f"Đã xảy ra lỗi: {e}")

finally:
    input("Nhấn Enter để đóng trình duyệt...")  # Giữ trình duyệt mở để kiểm tra
    driver.quit()
