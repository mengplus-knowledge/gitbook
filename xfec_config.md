��Linux Debian�У�ʹ��XFCE����ʱ������ͨ������������XFCE��panel����ͨ���漰�޸�XFCE�������ļ���ʹ�� `xfconf-query`������ֱ�Ӹ������á����������ʹ������������XFCE panel�Ĳ��裺

### 1. ʹ�� `xfconf-query` ����

`xfconf-query` ��XFCE����ϵͳ�������нӿڹ��ߣ�����������ѯ������XFCE�ĸ������á�

#### �鿴��ǰ��panel����

���ȣ�����Բ鿴��ǰpanel�����ã�

```bash
xfconf-query -c xfce4-panel -l
```

�⽫�г�������panel��ص������

#### ��ȡ�ض��������ֵ

��������ȡĳ���ض��������ֵ������ʹ���������

```bash
xfconf-query -c xfce4-panel -p /panels/panel-1/position
```

�⽫����panel-1��λ����Ϣ��

#### �����������ֵ

�����ʹ�� `xfconf-query`������panel�����á����磬����������panel��λ�ã�

```bash
xfconf-query -c xfce4-panel -p /panels/panel-1/position -s "bottom"
```

�⽫��panel-1��λ������Ϊ��Ļ�ײ���

#### ����µ�������

�����Ҫ����µ����������ʹ���������

```bash
xfconf-query -c xfce4-panel -p /panels/panel-1/new-property -t string -s "value"
```

�⽫Ϊpanel-1���һ���µ������

### 2. ֱ�ӱ༭�����ļ�

����ʹ�� `xfconf-query`���㻹����ֱ�ӱ༭XFCE�������ļ�����Щ�ļ�ͨ��λ�� `~/.config/xfce4/xfconf/xfce-perchannel-xml/`Ŀ¼�¡�

���磬Ҫ�༭panel�����ã������ҵ� `xfce4-panel.xml`�ļ���

```bash
nano ~/.config/xfce4/xfconf/xfce-perchannel-xml/xfce4-panel.xml
```

������ļ��У�������ֶ��޸�panel�����á�

### 3. ����PanelӦ�õ�����

�ڸ������ú��������Ҫ����XFCE panel��Ӧ�ø��ģ�

```bash
xfce4-panel --restart
```

### 4. ע������

- �ڱ༭�����ļ���ʹ�� `xfconf-query`ʱ����ȷ���˽��������������ã�����Ӱ��ϵͳ�ȶ��ԡ�
- �������޸������ļ�֮ǰ�ȱ���ԭʼ�ļ���

ͨ�������������������Ч��ʹ�����������ú͹���XFCE���滷���е�panel��
