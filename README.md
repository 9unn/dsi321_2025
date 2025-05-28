# EGAT Real-time Power Generation Scraper

สคริปต์ Python นี้ทำหน้าที่ดึงข้อมูลการผลิตไฟฟ้าแบบเรียลไทม์จากเว็บไซต์การไฟฟ้าฝ่ายผลิตแห่งประเทศไทย (กฟผ.) ที่ URL `https://www.sothailand.com/sysgen/egat/` โดยใช้ Selenium ในการโต้ตอบกับหน้าเว็บ และดึงข้อมูลจาก Console Log ของเบราว์เซอร์ ซึ่งเป็นที่ที่เว็บไซต์อัปเดตข้อมูลแบบไดนามิก ข้อมูลที่ดึงมาได้จะถูกบันทึกลงในไฟล์ CSV

สคริปต์ถูกออกแบบมาให้สามารถทำงานได้อย่างต่อเนื่อง เพื่อเก็บข้อมูลใหม่ตามช่วงเวลาที่กำหนด


## Structure

```
DSI321_2025/
├── _pycache_/
│   └── egat_pipeline.cpython-312.pyc
├── parquet/
│   └── egat_realtime_power_history.parquet
├── test-scraping/
│   ├── .ipynb_checkpoints/
│   ├── Dockerfile
│   ├── requirements.txt                         # Project dependencies
│   └── run_scraper_and_save_to_lakefs.ipynb     # Your original notebook for reference
├── UI/
│   └── streamlit_app.py
├── docker-compose.yml                           # Existing docker-compose
├── egat_pipeline.py
├── egat_realtime_power.csv
├── prefect.yaml
└── README.md
```


## คุณสมบัติเด่น

**Real-time Monitorin**: ดึงข้อมูลการผลิตไฟฟ้าจากระบบของ กฟผ. (EGAT) โดยตรง ทำให้สามารถเห็นสถานะของโครงข่ายไฟฟ้าของประเทศไทยได้ทันที
**Automated Data Collectio**: ลดความจำเป็นในการเก็บข้อมูลด้วยตนเอง โดยใช้การดึงข้อมูลอัตโนมัติในช่วงเวลาที่สามารถกำหนดได้ ทำให้ได้ชุดข้อมูลประวัติที่สม่ำเสมอ
**Predictive Analytics**: ใช้ forcasting model เพื่อคาดการณ์แนวโน้มความต้องการใช้ไฟฟ้า ช่วยให้สามารถวางแผนและจัดสรรทรัพยากรได้อย่างมีประสิทธิภาพ
**Interactive Visualization**: แสดงข้อมูลการผลิตไฟฟ้าที่ซับซ้อนผ่านอินเทอร์เฟซที่ใช้งานง่าย ทำให้ทั้งผู้ใช้ที่มีความรู้ด้านเทคนิคและไม่มีพื้นฐานสามารถเข้าใจข้อมูลได้
**Technical Innovation**: ใช้ Selenium เพื่อดึงข้อมูลจาก log ของเบราว์เซอร์แบบ dynamic แสดงถึงเทคนิคการดึงข้อมูลจากเว็บไซต์ที่ใช้ JavaScript
**Historical Analysis**: สร้างชุดข้อมูลแบบอนุกรมเวลา (time-series) ที่สมบูรณ์ เหมาะสำหรับการวิเคราะห์แนวโน้มและการตรวจจับความผิดปกติในการผลิตไฟฟ้า

## หลักการทำงาน

เว็บไซต์ `https://www.sothailand.com/sysgen/egat/` ดูเหมือนจะอัปเดตข้อมูลที่แสดงผล (กำลังการผลิตปัจจุบันหน่วยเป็น MW และอุณหภูมิ) ผ่าน JavaScript ซึ่ง JavaScript นี้ก็จะส่งข้อความการอัปเดตไปยัง Console ของเบราว์เซอร์ด้วย สคริปต์นี้ใช้ประโยชน์จากพฤติกรรมดังกล่าว:

1. **เริ่มต้น WebDriver:** ตั้งค่าและเปิดเบราว์เซอร์ Chrome ในโหมด Headless ผ่าน Selenium และเปิดใช้งาน `goog:loggingPrefs` เพื่อเก็บ Log จาก Console ของเบราว์เซอร์
2. **เปิด URL เป้าหมาย:** เปิดหน้าเว็บ กฟผ. ที่ระบุ
3. **รอข้อมูล:** หยุดรอสักครู่เพื่อให้หน้าเว็บโหลดสมบูรณ์และ JavaScript ทำงาน
4. **ดึงข้อมูลจาก Console:** เรียกดู Log จาก Console ของเบราว์เซอร์ และค้นหาข้อความที่มีรูปแบบ `updateMessageArea:`
5. **แยกส่วนข้อมูล:** ใช้ Regular Expression เพื่อแยกส่วนข้อมูลที่เกี่ยวข้อง (เช่น รหัสวันที่, เวลา, ค่า MW ปัจจุบัน, อุณหภูมิ) ออกจากข้อความ Log ที่ตรงกัน
6. **บันทึกลง CSV:** จัดรูปแบบข้อมูลที่ดึงมาได้พร้อมกับ `scrape_time` (เวลาที่ดึงข้อมูล) ให้อยู่ในรูป Dictionary และบันทึกเป็นแถวใหม่ต่อท้ายไฟล์ CSV ที่กำหนด
7. **ทำงานวนซ้ำ (ทางเลือก):** หากมีการเรียกใช้ `scrape_continuously` สคริปต์จะทำซ้ำขั้นตอนที่ 2-6 ตามช่วงเวลาที่กำหนด
8. **ปิด WebDriver:** ปิดการทำงานของเบราว์เซอร์ Driver อย่างถูกต้องเมื่อสคริปต์ทำงานเสร็จหรือถูกขัดจังหวะ

## Resources

- **Tools Used**
    - **Web Scraping**: Python `webdriver_manager` `Selenium`
    - **Data Validation**: `Pydantic`
    - **Data Storage**: `lakeFS`
    - **Orchestration**: `Prefect`
    - **Visualization**: `Streamlit`
    - **CI/CD**: GitHub Actions

- **Hardware Requirements**
    - Docker-compatible environment
    - Local or cloud system with:
        - At least 4 GB RAM
        - Internet access for EGAT web
        - Port availability for Prefect UI (default: `localhost:4200`)

## การติดตั้งและตั้งค่า

1- **Setup Steps**
    - Create a virtual environment
    ```bash
    python -m venv .venv
    ```
    - Install required packages
    ```bash
    #Windows config
    pip install -r test_scraping\requirements.txt
    ```
    ```bash
    #mac-os
    pip install -r test_scraping\requirements.txt
    ```
    - Activate the virtual environment
    ```bash
    #Windows config
    source .venv/Scripts/activate
    ```
    ```bash
    #mac-os
    source .venv/bin/activate
    ```

## Running Prefect

- Create a deployment for build, push, and pull for building Docker images, pushing code to the Docker registry, and pulling code to run the flow.
- **If there is a PREFECT_API_URL bug, run this script first.**
    ```bash
    #Windows config
    $env:PREFECT_API_URL = "http://127.0.0.1:4200/api"
    ```
    ```bash
    #mac-os
    export PREFECT_API_URL="http://127.0.0.1:4200/api"
    ```
- Deployments allow you to run flows on a schedule and trigger runs based on events.
    ```bash
    prefect deploy
    ```
- Start Prefect Worker to start pulling jobs from the Work Pool and Work Queue.
    ```bash
    pefect worker start --pool 'default-agent-pool' --work-queue 'default'
    ```
- Run Streamlit UI
    ```bash
    streamlit run UI/streamlit_app.py
    ```