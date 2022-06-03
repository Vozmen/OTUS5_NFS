# OTUS5_NFS
Homework

�������� �����:
- `vagrant up` ������ ��������� 2 ����������� ����������� ������
(������ NFS � �������) ��� �������������� ������ ���?�����?;
- �� ������� NFS ������ ���� ������������ � ��������������
����������;
- � ����������������? ���������� ������ ���� �������������
� ������ __upload__ � ������� �� ������ � ���?;
- ���������������� ���������� ������ ������������� �������������
�� ������� ��� ������ �����������? ������ (systemd, autofs ��� fstab -
����� ��������);
- ������������ � ������ NFS �� ������� ������ ���� ������������
� �������������� NFSv3 �� ��������� UDP;
- firewall ������ ���� ������� � �������� ��� �� �������,
��� � �� �������.

����������:

������:
��������� ����������� ��
yum install nfs-utils

������� firewall � ��������, ��� �� �������
systemctl enable filewalld --now

��� ������ ����������� �������� ����� firewall � ������������ ���
firewall-cmd --add-service="nfs3" \
--add-service="rpc-bind" \
--add-service="mountd" \
--permanent
firewall-cmd --reload

������� nfs ������
systemctl enable nfs --now

�������� �������� �����
ss -tnplu

������ ����� /srv/share/upload, ������ ��������� � ����� �� ��
mkdir -p /srv/share/upload
chown -R nfsnobody:nfsnobody /srv/share
chmod 0777 /srv/share/upload

� ����� /etc/exports ������, ����� ����� ����� ���������
cat << EOF > /etc/exports
/srv/share 192.168.50.11/32(rw,sync,root_squash)
EOF

������������� ���������� /srv/share/upload
exportfs -r

������:
��������� ����������� ��
yum install nfs-utils

������� firewall � ��������, ��� �� �������
systemctl enable filewalld --now

������� /etc/fstab ��� ��������������� ������������
echo "192.168.50.10:/srv/share/ /mnt nfs vers=3,proto=udp,noauto,x-
systemd.automount 0 0" >> /etc/fstab

������������ ������ ��� ���������������� ����������
systemctl daemon-reload
systemctl restart remote-fs.target

�������� �������� ������������
mount | grep mnt

������ � ������ ������������, ������ �� ����� �� ������� � �������. ��� ����� ��� ��������� �����.


��������� ��� ������
vagrant destroy -f

�������� Vagrantfile ��������� ��� ������� shell ������� ��� ������ �������
nfss.vm.provision "shell", path: "nfss_script.sh"
nfsc.vm.provision "shell", path: "nfsc_script.sh"

������ ������� � ���������������� ���������� � ����������� � ������� � ��� ����������� �������.
�������� vagrant up �������� ��� ������, ����� ������� �������� ��������� ��������.