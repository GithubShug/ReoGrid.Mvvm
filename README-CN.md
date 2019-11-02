
������һ��ͼ����Ϣ�ļ���Ŀ��ʾ���ʹ�� `ReoGrid.Mvvm`. �����������Ŀ `ReoGrid.Mvvm.Demo`.

![all.gif](https://i.loli.net/2019/10/23/RnLb5wEFKOJsd4c.gif)
<!--more-->
#### 1. ����һ��WPF��Ŀ.
#### 2. NuGet��װ ReoGrid.Mvvm

```
Install-Package ReoGrid.Mvvm
```
#### 3. ����һ��ͼ���ģ�ͣ�Model��

```cs
[WorksheetAttribute(Title = "Books")]
public class Book: IRecordModel
{
    [ColumnHeader(Index = 10, IsVisible = false)]
    public int Id { get; set; }

    [ColumnHeader(Index = 20, Text = "Name", Width = 150)]
    public string Title { get; set; }

    [ColumnHeader(Index = 30)]
    public string Author { get; set; }

    [ColumnHeader(Index = 35, Text = "Type")]
    public BindingType BindingType { get; set; }

    [ColumnHeader(Index = 36, Text = "OnSale")]
    public bool IsOnSale { get; set; }

    [NumberFormat(DecimalPlaces = 2)]
    [ColumnHeader(Index = 40)]
    public decimal Price { get; set; }

    [DateTimeFormat( CultureName = "en-US")]
    [ColumnHeader(Index = 45, Text = "Publish Date", Width = 200)]
    public DateTime Pubdate { get; set; }

    public int RowIndex { get; set; }
}

```

**(1) Model ����ʵ��`IRecordModel`�ӿ�**

`IRecordModel` ֻ��һ�� `RowIndex` ����, ����ȫ���ù�������ԣ����� `ReoGrid.Mvvm` �ڲ��õ��ġ�

**(2) `WorksheetAttribute` ����˵�������������**

��ѡ����ָ�������ԣ���ô����Model���������Ϊ���������ơ�

**(3) `ColumnHeader`������, ����ָ�� `Index` ���ԣ��������ǿ�ѡ�ġ�**

**(4) `DateTimeFormat` `DateTimeFormat` Ŀǰ������ʹ��**

`ReoGrid`����û������ʵ����Щ���ԡ���Ȼ��Ҳ�п������������

#### 4. ��ViewModel����Ҫ���޸�:

##### 4.1 ����������Ա����
```cs
private ObservableCollection<IRecordModel> _Books;
private WorksheetModel _WorksheetModel;
```
##### 4.2 ��ʼ��
```cs
_Books = new ObservableCollection<IRecordModel>();
for (int i = 0; i < 10; i++)
{
    Book book = new Book();
    book.Id = i;
    book.Title = string.Format("Title {0}", i);
    book.Author = string.Format("Author {0}", i);
    book.BindingType = BindingType.Hardback;
    book.IsOnSale = true;
    book.Price = (decimal)(i * 10.1);
    book.Pubdate = DateTime.Now;
    _Books.Add(book);
}
// ���� reoGridControl �� ReoGridControl �Ŀؼ�Ԫ��ʵ��
_WorksheetModel = new WorksheetModel(reoGridControl, typeof(Book), _Books);
//�����Ҫ������ֵǰ����������Ч�ԣ���ô��ʵ�ָú���
_WorksheetModel.OnBeforeChangeRecord += WorksheetModel_OnBeforeChangeRecord;
```
##### 4.3 �� `WorksheetModel_OnBeforeChangeRecord` ��������ʾ����ֵ��Ч�Լ��

```cs
private bool? WorksheetModel_OnBeforeChangeRecord(IRecordModel record, System.Reflection.PropertyInfo propertyInfo, object newProperyValue)
{
    if (propertyInfo.Name.Equals("Price"))
    {
        decimal price = Convert.ToDecimal(newProperyValue);
        if (price > 100m) //�������۸���100
        {
            MessageBox.Show("���۸��� 100�� ���������룡.", "Alert", MessageBoxButton.OK, MessageBoxImage.Warning);
            return true; // ���� true ��ȡ����������
        }
    }

    return null;
}
```
##### 4.4 ���ӡ�ɾ�����ƶ����༭ ģ�ͣ�Model��

```cs
// ����һ����Ŀ��Ϣ
int count = _Books.Count;
Book book = new Book();
book.Id = count;
book.Title = string.Format("Title {0}", count);
book.Author = string.Format("Author {0}", count);
book.BindingType = BindingType.Hardback;
book.IsOnSale = true;
book.Price = (decimal)(count * 10.11) > 100m ? 100m :(decimal)(count * 10.11);
book.Pubdate = DateTime.Now;
_Books.Add(book);

// �Ƴ�һ����Ŀ��Ϣ
if (_Books.Count > 0)
{
    _Books.RemoveAt(_Books.Count - 1);
}

// �ƶ�һ����Ŀ��Ϣ
if (_Books.Count > 2)
{
    _Books.Move(0, _Books.Count - 1);
}

// �༭һ����Ŀ��Ϣ
(_Books[0] as Book).Price = new Random(DateTime.Now.Millisecond).Next(1,100);
_WorksheetModel.UpadteRecord(_Books[0]); // �༭�� ģ�ͣ�Model�� ֮��Ҫ���� UpadteRecord ������ģ�ͣ�Model���ı仯ͬ������ͼ��View����
```