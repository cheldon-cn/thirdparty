# Introduction
Welcome to the snippet world .
this is the snippet code

- [memManager](https://github.com/cheldon-cn/): a memory manager 

- [x86_64-w64-mingw32 ](https://github.com/cheldon-cn/): configure items for different host
  
## 1. memManager

```
#include <vector>

//memory array
class CMemArray
{
public:
	CMemArray(){}
	~CMemArray(){}

public:
	int Add(char * pItem){
		m_list.push_back(pItem);
	}
	long Size(){
		return m_list.size();
	}
	int Set(long index, char* pItem){
		m_list.insert(index,pItem);
	}
	char* operator[](int index){
		return m_list[index];
	}

private:
	std::vector<char *> m_list;
};

//memory manager
class CMemManager
{
public:
	CMemManager();
	~CMemManager();

public:
	void *Alloc(int size);      //allocate buffer
	long Free();				//re goback to the start pos of the buffer 

private:
	CMemArray<char *>   m_list;
	char			   *m_buf;
	long				m_idx;
	int				    m_pos;
};


#define MEMBLOCKSIZE	(1024*1024*10)	//10M space
CMemManager::CMemManager(void)
{
	m_buf = NULL;
	m_pos = 0;        //record the pos of each memory block
	m_idx = 0;        //record the index of each memory block
}

CMemManager::~CMemManager(void)
{
	for (long i = 0; i < m_list.Size(); i++)
	{
		delete[]m_list[i];
	}
}

long CMemManager::Free()
{
	if (m_buf == NULL)return(1);

	m_buf = m_list[0];
	m_pos = 0;
	m_idx = 0;
	return(1);
}

void *CMemManager::Alloc(int size)
{
	if (m_buf == NULL)
	{
		m_buf = new char[MEMBLOCKSIZE];
		m_list.Add(m_buf);
	}

	// for linux or android,the memory buffer start position must be aligned, the default aligned byte is 4;
#if defined(BUTTER_LINUX) || defined(LINUX_ANDROID) || defined(BUTTER_IOS)
	if ((size % 4) == 0)
	{
		m_pos = (m_pos + 3) / 4 * 4;
	}
	else if ((size % 2) == 0)
	{
		m_pos = (m_pos + 1) / 2 * 2;
	}
#endif

	//m_buf
	if (MEMBLOCKSIZE < size + m_pos)
	{
		m_idx++;
		if (m_idx >= m_list.Size())
		{
			m_buf = new char[((MEMBLOCKSIZE > size) ? MEMBLOCKSIZE : size)];
			m_list.Add(m_buf);
		}
		else if (MEMBLOCKSIZE >= size)
			m_buf = m_list[m_idx];
		else
		{
			delete[]m_list[m_idx];
			m_buf = new char[size];
			m_list.Set(m_idx, m_buf);
		}
		m_pos = 0;
	}

	//allocate
	void *rtn = m_buf + m_pos;
	m_pos += size;
	return(rtn);
}

```

## 2. --host=x86_64-w64-mingw32 

```
./configure --host=i686-w64-mingw32     --prefix=/f/Codes/openSource/githubmaster/libiconv1.16/libiconv             CC=i686-w64-mingw32-gcc             CPPFLAGS="-I/usr/local/mingw32/include -Wall"             LDFLAGS="-L/usr/local/mingw32/lib"
```

```
PATH=/usr/local/mingw64/bin:$PATH
export PATH

./configure --host=x86_64-w64-mingw32 --prefix=/cygdrive/f/Codes/openSource/githubmaster/libiconv1.16/libiconv \
	CC=x86_64-w64-mingw32-gcc \
	CPPFLAGS="-I/usr/local/mingw64/include -Wall" \
	LDFLAGS="-L/usr/local/mingw64/lib"
	
```

## 3. arcgisonline url

https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/1/1/1

from  

https://github.com/giswqs/qgis-earthengine-examples/blob/master/Folium/ee-api-folium-setup.ipynb

```
from 

https://github.com/giswqs/qgis-earthengine-examples/blob/master/Basemaps/qgis_basemaps.py

sources = []
sources.append(["connections-xyz","Google Maps","","","","https://mt1.google.com/vt/lyrs=m&x=%7Bx%7D&y=%7By%7D&z=%7Bz%7D","","19","0"])
sources.append(["connections-xyz","Google Satellite", "", "", "", "https://mt1.google.com/vt/lyrs=s&x=%7Bx%7D&y=%7By%7D&z=%7Bz%7D", "", "19", "0"])
sources.append(["connections-xyz","Google Terrain", "", "", "", "https://mt1.google.com/vt/lyrs=t&x=%7Bx%7D&y=%7By%7D&z=%7Bz%7D", "", "19", "0"])
sources.append(["connections-xyz","Google Terrain Hybrid", "", "", "", "https://mt1.google.com/vt/lyrs=p&x=%7Bx%7D&y=%7By%7D&z=%7Bz%7D", "", "19", "0"])
sources.append(["connections-xyz","Google Satellite Hybrid", "", "", "", "https://mt1.google.com/vt/lyrs=y&x=%7Bx%7D&y=%7By%7D&z=%7Bz%7D", "", "19", "0"])
sources.append(["connections-xyz","Stamen Terrain", "", "", "Map tiles by Stamen Design, under CC BY 3.0. Data by OpenStreetMap, under ODbL", "http://tile.stamen.com/terrain/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "20", "0"])
sources.append(["connections-xyz","Stamen Toner", "", "", "Map tiles by Stamen Design, under CC BY 3.0. Data by OpenStreetMap, under ODbL", "http://tile.stamen.com/toner/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "20", "0"])
sources.append(["connections-xyz","Stamen Toner Light", "", "", "Map tiles by Stamen Design, under CC BY 3.0. Data by OpenStreetMap, under ODbL", "http://tile.stamen.com/toner-lite/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "20", "0"])
sources.append(["connections-xyz","Stamen Watercolor", "", "", "Map tiles by Stamen Design, under CC BY 3.0. Data by OpenStreetMap, under ODbL", "http://tile.stamen.com/watercolor/%7Bz%7D/%7Bx%7D/%7By%7D.jpg", "", "18", "0"])
sources.append(["connections-xyz","Wikimedia Map", "", "", "OpenStreetMap contributors, under ODbL", "https://maps.wikimedia.org/osm-intl/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "20", "1"])
sources.append(["connections-xyz","Wikimedia Hike Bike Map", "", "", "OpenStreetMap contributors, under ODbL", "http://tiles.wmflabs.org/hikebike/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "17", "1"])
sources.append(["connections-xyz","Esri Boundaries Places", "", "", "", "https://server.arcgisonline.com/ArcGIS/rest/services/Reference/World_Boundaries_and_Places/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "20", "0"])
sources.append(["connections-xyz","Esri Gray (dark)", "", "", "", "http://services.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Dark_Gray_Base/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "16", "0"])
sources.append(["connections-xyz","Esri Gray (light)", "", "", "", "http://services.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Light_Gray_Base/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "16", "0"])
sources.append(["connections-xyz","Esri National Geographic", "", "", "", "http://services.arcgisonline.com/ArcGIS/rest/services/NatGeo_World_Map/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "12", "0"])
sources.append(["connections-xyz","Esri Ocean", "", "", "", "https://services.arcgisonline.com/ArcGIS/rest/services/Ocean/World_Ocean_Base/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "10", "0"])
sources.append(["connections-xyz","Esri Satellite", "", "", "", "https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "17", "0"])
sources.append(["connections-xyz","Esri Standard", "", "", "", "https://server.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "17", "0"])
sources.append(["connections-xyz","Esri Terrain", "", "", "", "https://server.arcgisonline.com/ArcGIS/rest/services/World_Terrain_Base/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "13", "0"])
sources.append(["connections-xyz","Esri Transportation", "", "", "", "https://server.arcgisonline.com/ArcGIS/rest/services/Reference/World_Transportation/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "20", "0"])
sources.append(["connections-xyz","Esri Topo World", "", "", "", "http://services.arcgisonline.com/ArcGIS/rest/services/World_Topo_Map/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "20", "0"])
sources.append(["connections-xyz","OpenStreetMap Standard", "", "", "OpenStreetMap contributors, CC-BY-SA", "http://tile.openstreetmap.org/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "19", "0"])
sources.append(["connections-xyz","OpenStreetMap H.O.T.", "", "", "OpenStreetMap contributors, CC-BY-SA", "http://tile.openstreetmap.fr/hot/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "19", "0"])
sources.append(["connections-xyz","OpenStreetMap Monochrome", "", "", "OpenStreetMap contributors, CC-BY-SA", "http://tiles.wmflabs.org/bw-mapnik/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "19", "0"])
sources.append(["connections-xyz","Strava All", "", "", "OpenStreetMap contributors, CC-BY-SA", "https://heatmap-external-b.strava.com/tiles/all/bluered/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "15", "0"])
sources.append(["connections-xyz","Strava Run", "", "", "OpenStreetMap contributors, CC-BY-SA", "https://heatmap-external-b.strava.com/tiles/run/bluered/%7Bz%7D/%7Bx%7D/%7By%7D.png?v=19", "", "15", "0"])
sources.append(["connections-xyz","Open Weather Map Temperature", "", "", "Map tiles by OpenWeatherMap, under CC BY-SA 4.0", "http://tile.openweathermap.org/map/temp_new/%7Bz%7D/%7Bx%7D/%7By%7D.png?APPID=1c3e4ef8e25596946ee1f3846b53218a", "", "19", "0"])
sources.append(["connections-xyz","Open Weather Map Clouds", "", "", "Map tiles by OpenWeatherMap, under CC BY-SA 4.0", "http://tile.openweathermap.org/map/clouds_new/%7Bz%7D/%7Bx%7D/%7By%7D.png?APPID=ef3c5137f6c31db50c4c6f1ce4e7e9dd", "", "19", "0"])
sources.append(["connections-xyz","Open Weather Map Wind Speed", "", "", "Map tiles by OpenWeatherMap, under CC BY-SA 4.0", "http://tile.openweathermap.org/map/wind_new/%7Bz%7D/%7Bx%7D/%7By%7D.png?APPID=f9d0069aa69438d52276ae25c1ee9893", "", "19", "0"])
sources.append(["connections-xyz","CartoDb Dark Matter", "", "", "Map tiles by CartoDB, under CC BY 3.0. Data by OpenStreetMap, under ODbL.", "http://basemaps.cartocdn.com/dark_all/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "20", "0"])
sources.append(["connections-xyz","CartoDb Positron", "", "", "Map tiles by CartoDB, under CC BY 3.0. Data by OpenStreetMap, under ODbL.", "http://basemaps.cartocdn.com/light_all/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "20", "0"])
sources.append(["connections-xyz","Bing VirtualEarth", "", "", "", "http://ecn.t3.tiles.virtualearth.net/tiles/a{q}.jpeg?g=1", "", "19", "1"])

```

## 4.  北纬N：30°29′13.38″  东经E：114°28′49.83″

  中心点经纬度：30.487051085698166, 114.48050921768188  北纬N：30°29′13.38″  东经E：114°28′49.83″

## 5. GraphicsContext
```
public class LayoutControl extends Control {
    private static final int DEFAULT_WIDTH = 512;
    private static final int DEFAULT_HEIGHT = 512;
    private LayoutTool layoutTool = null;
    private Layout mLayout = null;
    private ILayoutSizeChangeListener mSizeChageListener = this::onSizeChanged;
    private ILayoutSelectSetChangeListener mSelectSetChangeListener = this::onSelectSetChanged;
    //endregion

    //region 构造
    public LayoutControl() {

        zoomCanvas = new javafx.scene.canvas.Canvas(DEFAULT_WIDTH, DEFAULT_HEIGHT);
        this.getChildren().add(zoomCanvas);
        GraphicsContext gczoom = zoomCanvas.getGraphicsContext2D();
        gczoom.setFill(javafx.scene.paint.Color.rgb(200, 200, 200, 0.4));
        gczoom.setStroke(javafx.scene.paint.Color.rgb(100, 110, 110, 1));
        gczoom.setLineWidth(0.02);
        zoomCanvas.setVisible(false);
        zoomCanvas.heightProperty().bind(this.heightProperty());
        zoomCanvas.widthProperty().bind(this.widthProperty());

        this.setPrefWidth(DEFAULT_WIDTH);
        this.setPrefHeight(DEFAULT_HEIGHT);
        LayoutControlNative.jni_OnSize(layoutControlNative, 0, DEFAULT_WIDTH, DEFAULT_HEIGHT);
        this.widthProperty().addListener((observable, oldValue, newValue) -> {
            double height = this.getHeight();
            if (height > 0.001 && newValue.intValue() > 0) {
                if (isFirstShow) {
                    isFirstShow = false;
                    Rect devRc = new Rect();
                    devRc.setXMin(0.0);
                    devRc.setYMin(0.0);
                    devRc.setXMax(newValue.doubleValue());
                    devRc.setYMax(height);
                    setDeviceRect(devRc);
                    setClientRect(devRc);
                    restoreWnd();
                } else {
                    LayoutControlNative.jni_OnSize(layoutControlNative, 0, newValue.intValue(), (int) height);
                }
            }
        });
        this.heightProperty().addListener((observable, oldValue, newValue) -> {
            double width = this.getWidth();
            if (width > 0.001 && newValue.intValue() > 0) {
                if (isFirstShow) {
                    isFirstShow = false;
                    Rect devRc = new Rect();
                    devRc.setXMin(0.0);
                    devRc.setYMin(0.0);
                    devRc.setXMax(width);
                    devRc.setYMax(newValue.doubleValue());
                    getTransformation().setDeviceRect(devRc);
                    getTransformation().setClientRect(devRc);
                    restoreWnd();
                } else {
                    LayoutControlNative.jni_OnSize(layoutControlNative, 0, (int) width, newValue.intValue());
                }
            }
        });

        this.setFocusTraversable(true);
    }

	
    public LayoutTool getLayoutTool() {
        return this.layoutTool;
    }

    private InteractionListener getInteractionListener() {
        return this.interactionListener;
    }

    private void setInteractionListener(InteractionListener interactionListener) {
        //Check.throwIfNull(interactionListener, "interactionListener");
        if (this.interactionListener != null) {
            this.interactionListener.onRemoved();
        }

        this.interactionListener = interactionListener;
        if (this.interactionListener == null)
            return;

        this.setOnMousePressed(this.interactionListener::onMousePressed);
        this.setOnMouseClicked(this.interactionListener::onMouseClicked);
        this.setOnMouseDragged(this.interactionListener::onMouseDragged);
        this.setOnMouseReleased(this.interactionListener::onMouseReleased);
        this.addEventFilter(MouseEvent.MOUSE_MOVED, this.interactionListener::onMouseMoved);
        this.setOnKeyPressed(this.interactionListener::onKeyPressed);
        this.setOnKeyReleased(this.interactionListener::onKeyReleased);
        this.setOnScroll(this.interactionListener::onScroll);
        this.setOnScrollStarted(this.interactionListener::onScrollStarted);
        this.setOnScrollFinished(this.interactionListener::onScrollFinished);
        this.interactionListener.onAdded();
    }

	
    public void setLayout(Layout layout) {
        if (this.mLayout != null) {
            this.mLayout.removeSizeChagedListener(mSizeChageListener);
            this.mLayout.removeSelectSetChangedListener(mSelectSetChangeListener);
        }

        this.mLayout = layout;
        if (layout == null)
            return;
        LayoutControlNative.jni_SetLayout(layoutControlNative, this.mLayout.getHandle());

        this.mLayout.addSizeChagedListener(mSizeChageListener);
        this.mLayout.addSelectSetChangedListener(mSelectSetChangeListener);

        this.layoutTool = new LayoutTool(this);
        this.setInteractionListener(layoutTool);
    }


    protected boolean isPointHit(MouseEvent event) {

        int  offsetx = (int) Math.abs(event.getX() - this.mLastPressX);
        int  offsety = (int) Math.abs(event.getY() - this.mLastPressY);

        boolean isPntClick = (offsetx <= 5 || offsety <= 5);
        return isPntClick;
    }

    public void onRefreshSelectRegion(MouseEvent event) {

        boolean isPntClick = isPointHit(event);
        if(isPntClick){
            bvalidDrag = false;
            return;
        }

        Bounds b = this.layoutControl.getBoundsInLocal();
        com.butter.geometry.Rect devRect = new com.butter.geometry.Rect();
        devRect.setXMin(Math.min(event.getX(), this.mLastPressX));
        devRect.setXMax(Math.max(event.getX(), this.mLastPressX));
        devRect.setYMin(b.getHeight() - Math.max(event.getY(), this.mLastPressY));
        devRect.setYMax(b.getHeight() - Math.min(event.getY(), this.mLastPressY));

        try {
            GraphicsContext gcc = this.layoutControl.getZoomCanvas().getGraphicsContext2D();
            gcc.clearRect(0, 0, this.layoutControl.getZoomCanvas().getWidth(), this.layoutControl.getZoomCanvas().getHeight());
            gcc.setLineDashes(0, 0);
            gcc.strokeRect(Math.min(event.getX(), this.mLastPressX), Math.min(event.getY(), this.mLastPressY), Math.abs(event.getX() - this.mLastPressX), Math.abs(event.getY() - this.mLastPressY));
            this.layoutControl.getZoomCanvas().setVisible(true);
            bvalidDrag = true;

        }catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

## 6. Interaction Listener
```

import javafx.scene.input.KeyEvent;
import javafx.scene.input.MouseEvent;
import javafx.scene.input.RotateEvent;
import javafx.scene.input.ScrollEvent;
import javafx.scene.input.SwipeEvent;
import javafx.scene.input.TouchEvent;
import javafx.scene.input.ZoomEvent;

public interface InteractionListener {
    default void onAdded() {
    }

    default void onRemoved() {
    }

    default void onMouseClicked(MouseEvent event) {
    }

    default void onMouseDragged(MouseEvent event) {
    }

    default void onMousePressed(MouseEvent event) {
    }

    default void onMouseReleased(MouseEvent event) {
    }

    default void onMouseMoved(MouseEvent event) {
    }

    default void onKeyPressed(KeyEvent event) {
    }

    default void onKeyReleased(KeyEvent event) {
    }

    default void onKeyTyped(KeyEvent event) {
    }

    default void onScroll(ScrollEvent event) {
    }

    default void onZoom(ZoomEvent event) {
    }

    default void onRotate(RotateEvent event) {
    }

    default void onMouseEntered(MouseEvent event) {
    }

    default void onMouseExited(MouseEvent event) {
    }

    default void onScrollStarted(ScrollEvent event) {
    }

    default void onScrollFinished(ScrollEvent event) {
    }

    default void onZoomStarted(ZoomEvent event) {
    }

    default void onZoomFinished(ZoomEvent event) {
    }

    default void onRotationStarted(RotateEvent event) {
    }

    default void onRotationFinished(RotateEvent event) {
    }
}
```

## 7. Callback


```
class ControlListener : public GListener
{
private:
	JavaVM* m_jvm;
	jobject m_jobj;
public:
	ControlListener(JavaVM* env, jobject obj)
	{
		m_jvm = env;
		m_jobj = obj;
	}
	virtual void OnFire(::EventType type, GEventArgs *args)
	{

		switch (type)
		{
		case MapViewAfterReFresh:
		{
			CGViewAfterReFreshArgs* refreshArgs = (CGViewAfterReFreshArgs*)args;
			JNIEnv* jniEnv = NULL;
			int state = m_jvm->GetEnv((void**)&jniEnv, 0x00010008);// JNI_VERSION_1_8);
			bool isNeedDetach = false;
			if (state != JNI_OK) {
				isNeedDetach = true;
#ifndef LINUX_ANDROID
				m_jvm->AttachCurrentThread((void**)&jniEnv, NULL);
#else
				m_jvm->AttachCurrentThread((JNIEnv**)&jniEnv, NULL);
#endif
			}
			CSmartFile* sFile = refreshArgs->GetImageBuffer();
			unsigned long dataLen = sFile->GetLength();
			jbyteArray data = jniEnv->NewByteArray(dataLen);

			char* ptBuf = new char[dataLen];

			sFile->SeekToBegin();
			sFile->Read(ptBuf, dataLen);
			jniEnv->SetByteArrayRegion(data, 0, dataLen, (jbyte*)ptBuf);
			safe_delete_array(ptBuf);

			jclass cls = jniEnv->GetObjectClass(m_jobj);
			jmethodID mid = jniEnv->GetMethodID(cls, "onCallbackAfterReFresh", "([B)V");
			jniEnv->CallVoidMethod(m_jobj, mid, data);
			jniEnv->DeleteLocalRef(data);
			jniEnv->DeleteLocalRef(cls);

			if (isNeedDetach)
				m_jvm->DetachCurrentThread();
		}
		break;
		case OverlayAfterRefresh:
		{
			CGViewAfterReFreshArgs* refreshArgs = (CGViewAfterReFreshArgs*)args;
			JNIEnv* jniEnv = NULL;
			int state = m_jvm->GetEnv((void**)&jniEnv, 0x00010008);// JNI_VERSION_1_8);
			bool isNeedDetach = false;
			if (state != JNI_OK) {
				isNeedDetach = true;
#ifndef LINUX_ANDROID
				m_jvm->AttachCurrentThread((void**)&jniEnv, NULL);
#else
				m_jvm->AttachCurrentThread((JNIEnv**)&jniEnv, NULL);
#endif
			}
			CSmartFile* sFile = refreshArgs->GetImageBuffer();
			unsigned long dataLen = sFile->GetLength();
			jbyteArray data = jniEnv->NewByteArray(dataLen);

			char* ptBuf = new char[dataLen];

			sFile->SeekToBegin();
			sFile->Read(ptBuf, dataLen);
			jniEnv->SetByteArrayRegion(data, 0, dataLen, (jbyte*)ptBuf);
			safe_delete_array(ptBuf);

			jclass cls = jniEnv->GetObjectClass(m_jobj);
			jmethodID mid = jniEnv->GetMethodID(cls, "onCallbackAfterOverlayReFresh", "([B)V");
			jniEnv->CallVoidMethod(m_jobj, mid, data);
			jniEnv->DeleteLocalRef(data);
			jniEnv->DeleteLocalRef(cls);

			if (isNeedDetach)
				m_jvm->DetachCurrentThread();
		}
		break;
		case dataViewBeginDrawing:
		{
			JNIEnv* jniEnv = NULL;
			int state = m_jvm->GetEnv((void**)&jniEnv, 0x00010008);// JNI_VERSION_1_8);
			bool isNeedDetach = false;
			if (state != JNI_OK) {
				isNeedDetach = true;
#ifndef LINUX_ANDROID
				m_jvm->AttachCurrentThread((void**)&jniEnv, NULL);
#else
				m_jvm->AttachCurrentThread((JNIEnv**)&jniEnv, NULL);
#endif
			}

			jclass cls = jniEnv->GetObjectClass(m_jobj);
			jmethodID mid = jniEnv->GetMethodID(cls, "onCallbackBeforeRefesh", "()V");
			jniEnv->CallVoidMethod(m_jobj, mid);
			jniEnv->DeleteLocalRef(cls);

			if (isNeedDetach)
				m_jvm->DetachCurrentThread();
		}
		break;
		default:
			break;
		}
	}
};
class CGMapViewX : public CGMapView
{
private:
	ControlListener* m_listenner;
	jobject m_job;
	JavaVM* m_jvm;
public:
	CGMapViewX(JNIEnv* env, jobject obj) :m_listenner(NULL)
	{
		env->GetJavaVM(&m_jvm);
		m_job = env->NewGlobalRef(obj);

		m_listenner = new ControlListener(m_jvm, m_job);
		this->Add(m_listenner, allEvent);
		this->SetImageBufferType(1);//bgra
	}
	virtual ~CGMapViewX()
	{
		delete m_listenner;

		JNIEnv* jniEnv = NULL;
		int state = m_jvm->GetEnv((void**)&jniEnv, 0x00010008);// JNI_VERSION_1_8);
		bool isNeedDetach = false;
		if (state != JNI_OK) {
			isNeedDetach = true;
#ifndef LINUX_ANDROID
			m_jvm->AttachCurrentThread((void**)&jniEnv, NULL);
#else
			m_jvm->AttachCurrentThread((JNIEnv**)&jniEnv, NULL);
#endif
		}
		jniEnv->DeleteGlobalRef(m_job);

		if (isNeedDetach)
			m_jvm->DetachCurrentThread();
	}
};

JNIEXPORT jlong JNICALL Java_com_butter_controls_ControlNative_jni_1CreateObj(JNIEnv *env, jclass jclassobj, jobject dataobj)
{
	CGMapViewX* pMapView = new CGMapViewX(env, dataobj);
	return (jlong)pMapView;
}
JNIEXPORT void JNICALL Java_com_butter_controls_ControlNative_jni_1DeleteObj(JNIEnv *env, jclass jclassobj, jlong handle)
{
	CGMapViewX* pMapView = (CGMapViewX*)handle;
	safe_delete(pMapView);
}


```
## 8. A dynamic link library (DLL) initialization routine failed

![initialization routine failed](./img/8_initialization_routine_failed.png)
```
use IntelliJ IDEA to debug some program, when load some dynamic link library (*.dll),
throw out some error infomation: "A dynamic link library (DLL) initialization routine failed" ;

when check the dll file ,the file exist and the located path has been set in the Computer PATH ENV,
besides this,use the dependency tool(dependency walker), we get the full complete dependent file list in the table view;

the method to solve this situation:

1. cmd --->regedit 
2. locate the target program node;
   eg: HKEY_CURRENT_USER\Software\Butter\AppProduct
3. delete the java node which set the java infomation,
   and node name make consist of the HEX code;


```



-----
Copyright 2020 - 2021 @ [cheldon](https://github.com/cheldon-cn/).
