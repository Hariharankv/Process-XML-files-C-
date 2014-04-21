Process-XML-files-C-
====================
using System;
using System.IO;
using System.Windows.Forms;
using System.Xml;
using System.Collections;

namespace XmlConvertor
{
    public partial class Form1 : Form
    {
        private int nCountieValLen;
        private ArrayList arlColoCodes;
        XmlDocument XmlDoc = null;
        XmlNode nodeXmlNode = null, subNodeXmlNode = null;
        XmlElement ptXmlElement = null;
        XmlAttribute latXmlAttribute = null, lngXmlAttribute = null, nameXmlAttribute = null, colorXmlAttribute = null;

        public Form1()
        {
            InitializeComponent();
            arlColoCodes = new ArrayList();
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            try
            {
                string strColorcodePath = AppDomain.CurrentDomain.BaseDirectory + "colorcodes.xml";
                XmlDocument docc = new XmlDocument();
                docc.Load(strColorcodePath);
                XmlNodeList nlist = docc.DocumentElement.GetElementsByTagName("color");
                for (int j = 0; j < nlist.Count; j++)
                {
                    string strColor = "";
                    XmlElement roott = docc.DocumentElement;
                    XmlElement resources = (XmlElement)roott.ChildNodes[j];
                    strColor = resources.InnerText;
                    //MessageBox.Show(strColor);
                    arlColoCodes.Add(strColor);   
                }
                
                string strFolder = "D:\\UScounties";

                DirectoryInfo varDirectoryInfo = new DirectoryInfo(strFolder);
                DirectoryInfo[] dirs = varDirectoryInfo.GetDirectories("*.*");

                string strFilePath = "";
                foreach (DirectoryInfo dir in dirs)
                {
                    strFilePath = AppDomain.CurrentDomain.BaseDirectory + "\\Files\\" + dir.ToString() + ".xml";
                    FileStream fs = new FileStream(strFilePath, FileMode.CreateNew, FileAccess.Write);
                    StreamWriter writer = new StreamWriter(fs);
                    writer.Write("<counties>\n</counties>");
                    writer.Close();
                    fs.Close();

                    getDirsFiles(dir, dir.ToString(), strFilePath);
                }
            }
            catch (Exception exMe)
            {
                MessageBox.Show(exMe.Message.ToString());
            }
        }

        private void getDirsFiles(DirectoryInfo varDirectoryInfo, string strDirectory, string strFilePath)
        {
            int nFileCnt = 0;
            XmlDoc = new XmlDocument();
            XmlDoc.Load(AppDomain.CurrentDomain.BaseDirectory + "\\counties.xml");
            
            string strFilter = "*.xml";
            string[] m_arExt = strFilter.Split(';');
            FileInfo[] files = null;

            foreach (string filter in m_arExt)
            {
                files = varDirectoryInfo.GetFiles(filter);
              
                    foreach (FileInfo file in files)
                    {
                        //string hend = "colour="#ff0000"";
                        //subNodeXmlNode = XmlDoc.CreateNode(XmlNodeType.Element, file.Name.Substring(0, file.Name.Length - 4), null);
                        subNodeXmlNode = XmlDoc.CreateNode(XmlNodeType.Element, "county", null);
                        nameXmlAttribute = XmlDoc.CreateAttribute("name");
                        colorXmlAttribute = XmlDoc.CreateAttribute("colour");

                        subNodeXmlNode.Attributes.Append(nameXmlAttribute);
                        subNodeXmlNode.Attributes.Append(colorXmlAttribute);

                        subNodeXmlNode.Attributes[0].Value = file.Name.Substring(0, file.Name.Length - 4);
                        //subNodeXmlNode.Attributes[1].Value = "#ff0000";

                        if (nFileCnt == 156)
                            nFileCnt = 0;

                        subNodeXmlNode.Attributes[1].Value = arlColoCodes[nFileCnt].ToString(); 

                        nCountieValLen = 0;
                        string[,] strLatLong = GetBoundariesOfCounties(file.FullName, file.Name, strDirectory);

                        if (strLatLong != null)
                        {
                            for (int i = 0; i < nCountieValLen; i++)
                            {
                                ptXmlElement = XmlDoc.CreateElement("pt");
                                subNodeXmlNode.AppendChild(ptXmlElement);

                                latXmlAttribute = XmlDoc.CreateAttribute("lat");
                                ptXmlElement.Attributes.Append(latXmlAttribute);
                                lngXmlAttribute = XmlDoc.CreateAttribute("lng");
                                ptXmlElement.Attributes.Append(lngXmlAttribute);
                                ptXmlElement.Attributes[0].Value = strLatLong[i, 0];
                                ptXmlElement.Attributes[1].Value = strLatLong[i, 1];
                            }
                            //node.AppendChild(subNode);
                            XmlDoc.DocumentElement.AppendChild(subNodeXmlNode);
                        }

                        nFileCnt = nFileCnt + 1;
                    }
                }
            
            XmlDoc.Save(strFilePath);
        }

        public string[,] GetBoundariesOfCounties(string strFileLocation, string strFileName, string strDirectory)
        {
            //try
            //{
            string[,] strLatLong = null;
            int i = 0;
            XmlDocument XmlDocTar = new XmlDocument();
            XmlDocTar.Load(strFileLocation);
            foreach (XmlElement xElement in XmlDocTar.DocumentElement)
            {
                nCountieValLen = xElement.ChildNodes.Count;
                strLatLong = new string[nCountieValLen, 2];
                foreach (XmlNode xNode in xElement.ChildNodes)
                {
                    strLatLong[i, 0] = xNode.Attributes[0].Value;
                    strLatLong[i, 1] = xNode.Attributes[1].Value;
                    i = i + 1;
                }
            }
            return strLatLong;
            //}
            //catch
            //{
            //    Directory.CreateDirectory("D:\\ErrorFiles\\" + strDirectory);
            //    File.Copy(strFileLocation, "D:\\ErrorFiles\\" + strDirectory + "\\" + strFileName);
            //    return null;
            //}
        }
    }
}
