# Controle-de-visitas-
import React, { useState, useEffect } from 'react';

// Main App Component
function App() {
  // --- State Management ---
  const [visits, setVisits] = useState(() => {
    // Load visits from local storage if available, otherwise use initial data
    const savedVisits = localStorage.getItem('visits');
    return savedVisits ? JSON.parse(savedVisits) : [];
  });

  const [projects, setProjects] = useState(() => {
    // Load projects from local storage or use initial data
    const savedProjects = localStorage.getItem('projects');
    return savedProjects ? JSON.parse(savedProjects) : [
      { id: 'proj1', name: 'Obra Central Park', address: 'Rua das Flores, 123, São Paulo', clientCompany: 'Construtora Sol', contactPerson: 'Ana Paula', contactPhone: '(11) 98765-4321', status: 'Em Andamento', startDate: '2024-01-15', endDate: '2025-12-31' },
      { id: 'proj2', name: 'Edifício Panorama', address: 'Av. Principal, 456, Rio de Janeiro', clientCompany: 'Incorporadora Céu', contactPerson: 'Carlos Alberto', contactPhone: '(21) 99876-5432', status: 'Em Andamento', startDate: '2023-08-01', endDate: '2024-11-30' },
      { id: 'proj3', name: 'Residencial Verde', address: 'Travessa da Mata, 789, Belo Horizonte', clientCompany: 'Urbanizadora Paz', contactPerson: 'Mariana Silva', contactPhone: '(31) 97654-3210', status: 'Concluída', startDate: '2022-03-10', endDate: '2023-05-20' },
    ];
  });

  const [contractors, setContractors] = useState(() => {
    // Load contractors from local storage or use initial data
    const savedContractors = localStorage.getItem('contractors');
    return savedContractors ? JSON.parse(savedContractors) : [
      { id: 'cont1', name: 'Construtora Alfa Ltda.', cnpj: '01.234.567/0001-89', mainContact: 'Roberto Gomes', phone: '(11) 2345-6789', email: 'contato@alfa.com.br' },
      { id: 'cont2', name: 'Engenharia Beta S.A.', cnpj: '98.765.432/0001-01', mainContact: 'Fernanda Lima', phone: '(21) 8765-4321', email: 'info@betaeng.com.br' },
      { id: 'cont3', name: 'Soluções Gamma Eireli', cnpj: '12.345.678/0001-23', mainContact: 'Gustavo Pereira', phone: '(31) 3456-7890', email: 'sac@gamma.com.br' },
    ];
  });

  const [activeTab, setActiveTab] = useState('visitLog'); // 'visitLog', 'projects', 'contractors', 'history'
  const [filterProject, setFilterProject] = useState('');
  const [filterContractor, setFilterContractor] = useState('');

  // Save data to local storage whenever it changes
  useEffect(() => {
    localStorage.setItem('visits', JSON.stringify(visits));
  }, [visits]);

  useEffect(() => {
    localStorage.setItem('projects', JSON.stringify(projects));
  }, [projects]);

  useEffect(() => {
    localStorage.setItem('contractors', JSON.stringify(contractors));
  }, [contractors]);

  // --- Form Data State ---
  const initialFormData = {
    date: new Date().toISOString().slice(0, 10),
    project: '',
    contractor: '',
    fiscal: '',
    numWorkers: '',
    ppeHelmet: 'Conforme',
    ppeGlasses: 'Conforme',
    ppeGloves: 'Conforme',
    ppeFootwear: 'Conforme',
    ppeSafetyBelt: 'Não Aplicável',
    ppeEarProtection: 'Conforme',
    otherPpe: '',
    generalObservations: '',
    nonConformitiesPpe: '',
    nonConformitiesGeneral: '',
    recommendedActions: '',
    correctionDeadline: '',
    status: 'Aberta',
    photosLink: '',
    followUpObservations: ''
  };
  const [formData, setFormData] = useState(initialFormData);

  // --- Handlers ---
  const handleInputChange = (e) => {
    const { name, value } = e.target;
    setFormData({ ...formData, [name]: value });
  };

  const handleSubmitVisit = (e) => {
    e.preventDefault();
    if (!formData.project || !formData.contractor || !formData.fiscal || !formData.date) {
      // Using a custom modal/alert for better UX instead of browser's alert()
      showCustomAlert('Por favor, preencha os campos obrigatórios: Obra, Empresa Executora, Fiscal Responsável e Data da Visita.');
      return;
    }

    const newVisit = {
      id: Date.now().toString(), // Unique ID for the visit
      ...formData
    };
    setVisits([...visits, newVisit]);
    setFormData(initialFormData); // Reset form
    showCustomAlert('Visita registrada com sucesso!', 'success');
  };

  const handleAddProject = (e) => {
    e.preventDefault();
    const newProjectName = prompt('Digite o nome da nova obra:');
    if (newProjectName && !projects.some(p => p.name === newProjectName)) {
      setProjects([...projects, { id: Date.now().toString(), name: newProjectName, address: '', clientCompany: '', contactPerson: '', contactPhone: '', status: 'Em Andamento', startDate: '', endDate: '' }]);
      showCustomAlert('Obra adicionada!');
    } else if (newProjectName) {
      showCustomAlert('Esta obra já existe.', 'error');
    }
  };

  const handleAddContractor = (e) => {
    e.preventDefault();
    const newContractorName = prompt('Digite o nome da nova empresa executora:');
    if (newContractorName && !contractors.some(c => c.name === newContractorName)) {
      setContractors([...contractors, { id: Date.now().toString(), name: newContractorName, cnpj: '', mainContact: '', phone: '', email: '' }]);
      showCustomAlert('Empresa executora adicionada!');
    } else if (newContractorName) {
      showCustomAlert('Esta empresa já existe.', 'error');
    }
  };

  // Custom Alert State and Component
  const [customAlert, setCustomAlert] = useState({ message: '', type: '', visible: false });

  const showCustomAlert = (message, type = 'info') => {
    setCustomAlert({ message, type, visible: true });
    setTimeout(() => {
      setCustomAlert({ ...customAlert, visible: false });
    }, 3000); // Hide after 3 seconds
  };

  const CustomAlert = () => {
    if (!customAlert.visible) return null;

    const bgColor = customAlert.type === 'success' ? 'bg-green-500' : customAlert.type === 'error' ? 'bg-red-500' : 'bg-blue-500';

    return (
      <div className={`fixed bottom-4 right-4 p-4 rounded-lg shadow-lg text-white ${bgColor} z-50 transition-opacity duration-300`}>
        {customAlert.message}
      </div>
    );
  };


  // --- Filtered Visits for History/List ---
  const filteredVisits = visits.filter(visit => {
    const matchesProject = filterProject ? visit.project === filterProject : true;
    const matchesContractor = filterContractor ? visit.contractor === filterContractor : true;
    return matchesProject && matchesContractor;
  });

  // --- Components for different tabs ---

  const VisitLogForm = () => (
    <div className="p-6 bg-white rounded-xl shadow-lg animate-fade-in">
      <h2 className="text-3xl font-extrabold mb-6 text-gray-900 text-center">Registrar Nova Visita Técnica</h2>
      <form onSubmit={handleSubmitVisit} className="grid grid-cols-1 md:grid-cols-2 gap-6">
        {/* Basic Info */}
        <div className="col-span-1">
          <label htmlFor="date" className="block text-sm font-semibold text-gray-700 mb-1">Data da Visita</label>
          <input type="date" id="date" name="date" value={formData.date} onChange={handleInputChange} required
            className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 transition duration-200 ease-in-out hover:border-indigo-400" />
        </div>
        <div className="col-span-1">
          <label htmlFor="project" className="block text-sm font-semibold text-gray-700 mb-1">Obra</label>
          <select id="project" name="project" value={formData.project} onChange={handleInputChange} required
            className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 bg-white transition duration-200 ease-in-out hover:border-indigo-400">
            <option value="">Selecione uma obra</option>
            {projects.map(proj => (
              <option key={proj.id} value={proj.name}>{proj.name}</option>
            ))}
          </select>
        </div>
        <div className="col-span-1">
          <label htmlFor="contractor" className="block text-sm font-semibold text-gray-700 mb-1">Empresa Executora</label>
          <select id="contractor" name="contractor" value={formData.contractor} onChange={handleInputChange} required
            className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 bg-white transition duration-200 ease-in-out hover:border-indigo-400">
            <option value="">Selecione uma empresa</option>
            {contractors.map(cont => (
              <option key={cont.id} value={cont.name}>{cont.name}</option>
            ))}
          </select>
        </div>
        <div className="col-span-1">
          <label htmlFor="fiscal" className="block text-sm font-semibold text-gray-700 mb-1">Fiscal Responsável</label>
          <input type="text" id="fiscal" name="fiscal" value={formData.fiscal} onChange={handleInputChange} required
            className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 transition duration-200 ease-in-out hover:border-indigo-400" />
        </div>
        <div className="col-span-1">
          <label htmlFor="numWorkers" className="block text-sm font-semibold text-gray-700 mb-1">Número de Funcionários Presentes</label>
          <input type="number" id="numWorkers" name="numWorkers" value={formData.numWorkers} onChange={handleInputChange}
            className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 transition duration-200 ease-in-out hover:border-indigo-400" />
        </div>

        {/* PPE Compliance */}
        <div className="md:col-span-2 mt-4 p-4 border border-gray-200 rounded-lg bg-gray-50">
          <h3 className="text-xl font-bold mb-4 text-gray-800">Conformidade de EPIs</h3>
          <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
            {['ppeHelmet', 'ppeGlasses', 'ppeGloves', 'ppeFootwear', 'ppeSafetyBelt', 'ppeEarProtection'].map(ppe => (
              <div key={ppe}>
                <label htmlFor={ppe} className="block text-sm font-medium text-gray-700 mb-1">
                  {ppe === 'ppeHelmet' && 'Capacete'}
                  {ppe === 'ppeGlasses' && 'Óculos de Segurança'}
                  {ppe === 'ppeGloves' && 'Luvas'}
                  {ppe === 'ppeFootwear' && 'Calçado de Segurança'}
                  {ppe === 'ppeSafetyBelt' && 'Cinto de Segurança (altura)'}
                  {ppe === 'ppeEarProtection' && 'Protetor Auricular'}
                </label>
                <select id={ppe} name={ppe} value={formData[ppe]} onChange={handleInputChange}
                  className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 bg-white transition duration-200 ease-in-out hover:border-indigo-400">
                  <option value="Conforme">Conforme</option>
                  <option value="Não Conforme">Não Conforme</option>
                  <option value="Não Aplicável">Não Aplicável</option>
                </select>
              </div>
            ))}
          </div>
          <div className="mt-4">
            <label htmlFor="otherPpe" className="block text-sm font-semibold text-gray-700 mb-1">Outros EPIs (Especificar)</label>
            <textarea id="otherPpe" name="otherPpe" value={formData.otherPpe} onChange={handleInputChange} rows="2"
              className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 transition duration-200 ease-in-out hover:border-indigo-400"></textarea>
          </div>
        </div>

        {/* Observations and Actions */}
        <div className="md:col-span-2 p-4 border border-gray-200 rounded-lg bg-gray-50">
          <h3 className="text-xl font-bold mb-4 text-gray-800">Observações e Ações</h3>
          <div className="grid grid-cols-1 gap-4">
            <div>
              <label htmlFor="generalObservations" className="block text-sm font-semibold text-gray-700 mb-1">Observações Gerais da Condição da Obra</label>
              <textarea id="generalObservations" name="generalObservations" value={formData.generalObservations} onChange={handleInputChange} rows="3"
                className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 transition duration-200 ease-in-out hover:border-indigo-400"></textarea>
            </div>
            <div>
              <label htmlFor="nonConformitiesPpe" className="block text-sm font-semibold text-gray-700 mb-1">Não Conformidades Identificadas (EPIs)</label>
              <textarea id="nonConformitiesPpe" name="nonConformitiesPpe" value={formData.nonConformitiesPpe} onChange={handleInputChange} rows="3"
                className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 transition duration-200 ease-in-out hover:border-indigo-400"></textarea>
            </div>
            <div>
              <label htmlFor="nonConformitiesGeneral" className="block text-sm font-semibold text-gray-700 mb-1">Não Conformidades Identificadas (Gerais)</label>
              <textarea id="nonConformitiesGeneral" name="nonConformitiesGeneral" value={formData.nonConformitiesGeneral} onChange={handleInputChange} rows="3"
                className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 transition duration-200 ease-in-out hover:border-indigo-400"></textarea>
            </div>
            <div>
              <label htmlFor="recommendedActions" className="block text-sm font-semibold text-gray-700 mb-1">Ações Recomendadas</label>
              <textarea id="recommendedActions" name="recommendedActions" value={formData.recommendedActions} onChange={handleInputChange} rows="3"
                className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 transition duration-200 ease-in-out hover:border-indigo-400"></textarea>
            </div>
          </div>
        </div>

        <div className="col-span-1">
          <label htmlFor="correctionDeadline" className="block text-sm font-semibold text-gray-700 mb-1">Prazo para Correção</label>
          <input type="date" id="correctionDeadline" name="correctionDeadline" value={formData.correctionDeadline} onChange={handleInputChange}
            className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 transition duration-200 ease-in-out hover:border-indigo-400" />
        </div>
        <div className="col-span-1">
          <label htmlFor="status" className="block text-sm font-semibold text-gray-700 mb-1">Status da Não Conformidade</label>
          <select id="status" name="status" value={formData.status} onChange={handleInputChange}
            className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 bg-white transition duration-200 ease-in-out hover:border-indigo-400">
            <option value="Aberta">Aberta</option>
            <option value="Em Andamento">Em Andamento</option>
            <option value="Fechada">Fechada</option>
          </select>
        </div>
        <div className="md:col-span-2">
          <label htmlFor="photosLink" className="block text-sm font-semibold text-gray-700 mb-1">Fotos (Links para Google Drive, etc.)</label>
          <input type="url" id="photosLink" name="photosLink" value={formData.photosLink} onChange={handleInputChange}
            className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 transition duration-200 ease-in-out hover:border-indigo-400" />
        </div>
        <div className="md:col-span-2">
          <label htmlFor="followUpObservations" className="block text-sm font-semibold text-gray-700 mb-1">Observações de Acompanhamento</label>
          <textarea id="followUpObservations" name="followUpObservations" value={formData.followUpObservations} onChange={handleInputChange} rows="3"
            className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 transition duration-200 ease-in-out hover:border-indigo-400"></textarea>
        </div>

        <div className="md:col-span-2 flex justify-end mt-6">
          <button type="submit"
            className="inline-flex items-center px-6 py-3 border border-transparent text-base font-medium rounded-full shadow-lg text-white bg-gradient-to-r from-indigo-600 to-purple-600 hover:from-indigo-700 hover:to-purple-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transition duration-300 ease-in-out transform hover:-translate-y-1">
            Registrar Visita
          </button>
        </div>
      </form>
    </div>
  );

  const VisitHistory = () => (
    <div className="p-6 bg-white rounded-xl shadow-lg animate-fade-in">
      <h2 className="text-3xl font-extrabold mb-6 text-gray-900 text-center">Histórico de Visitas</h2>

      {/* Filters */}
      <div className="mb-6 flex flex-col sm:flex-row gap-4 bg-gray-50 p-4 rounded-lg shadow-inner">
        <div className="flex-1">
          <label htmlFor="filterProject" className="block text-sm font-semibold text-gray-700 mb-1">Filtrar por Obra</label>
          <select id="filterProject" value={filterProject} onChange={(e) => setFilterProject(e.target.value)}
            className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 bg-white transition duration-200 ease-in-out hover:border-indigo-400">
            <option value="">Todas as Obras</option>
            {projects.map(proj => (
              <option key={proj.id} value={proj.name}>{proj.name}</option>
            ))}
          </select>
        </div>
        <div className="flex-1">
          <label htmlFor="filterContractor" className="block text-sm font-semibold text-gray-700 mb-1">Filtrar por Empresa Executora</label>
          <select id="filterContractor" value={filterContractor} onChange={(e) => setFilterContractor(e.target.value)}
            className="mt-1 block w-full rounded-lg border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 bg-white transition duration-200 ease-in-out hover:border-indigo-400">
            <option value="">Todas as Empresas</option>
            {contractors.map(cont => (
              <option key={cont.id} value={cont.name}>{cont.name}</option>
            ))}
          </select>
        </div>
      </div>

      {/* Visit Table */}
      {filteredVisits.length === 0 ? (
        <p className="text-center text-gray-500 text-lg py-8">Nenhuma visita encontrada com os filtros aplicados.</p>
      ) : (
        <div className="overflow-x-auto rounded-lg shadow-md border border-gray-200">
          <table className="min-w-full divide-y divide-gray-200">
            <thead className="bg-gradient-to-r from-blue-500 to-indigo-600 text-white">
              <tr>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider rounded-tl-lg">Data</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider">Obra</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider">Empresa</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider">Fiscal</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider">EPIs (Resumo)</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider">Não Conformidades</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider">Status</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider rounded-tr-lg">Prazo Correção</th>
              </tr>
            </thead>
            <tbody className="bg-white divide-y divide-gray-200">
              {filteredVisits.map((visit, index) => (
                <tr key={visit.id} className={index % 2 === 0 ? 'bg-white' : 'bg-gray-50'}>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{visit.date}</td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{visit.project}</td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{visit.contractor}</td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{visit.fiscal}</td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                    {visit.ppeHelmet === 'Não Conforme' && 'Capacete, '}
                    {visit.ppeGlasses === 'Não Conforme' && 'Óculos, '}
                    {visit.ppeGloves === 'Não Conforme' && 'Luvas, '}
                    {visit.ppeFootwear === 'Não Conforme' && 'Calçado, '}
                    {visit.ppeSafetyBelt === 'Não Conforme' && 'Cinto, '}
                    {visit.ppeEarProtection === 'Não Conforme' && 'Protetor Auricular'}
                    {(!visit.nonConformitiesPpe && !visit.nonConformitiesGeneral) && 'Conforme'}
                  </td>
                  <td className="px-6 py-4 text-sm text-gray-900 max-w-xs overflow-hidden text-ellipsis">
                    {visit.nonConformitiesPpe && `EPIs: ${visit.nonConformitiesPpe}`}
                    {visit.nonConformitiesPpe && visit.nonConformitiesGeneral && <br />}
                    {visit.nonConformitiesGeneral && `Gerais: ${visit.nonConformitiesGeneral}`}
                    {!visit.nonConformitiesPpe && !visit.nonConformitiesGeneral && 'Nenhuma'}
                  </td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm">
                    <span className={`px-2 inline-flex text-xs leading-5 font-semibold rounded-full
                      ${visit.status === 'Aberta' ? 'bg-red-100 text-red-800' :
                        visit.status === 'Em Andamento' ? 'bg-yellow-100 text-yellow-800' :
                        'bg-green-100 text-green-800'}`}>
                      {visit.status}
                    </span>
                  </td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{visit.correctionDeadline || 'N/A'}</td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      )}
    </div>
  );

  const ProjectsList = () => (
    <div className="p-6 bg-white rounded-xl shadow-lg animate-fade-in">
      <h2 className="text-3xl font-extrabold mb-6 text-gray-900 text-center">Obras Cadastradas</h2>
      <button onClick={handleAddProject}
        className="mb-6 inline-flex items-center px-6 py-3 border border-transparent text-base font-medium rounded-full shadow-lg text-white bg-gradient-to-r from-blue-500 to-sky-500 hover:from-blue-600 hover:to-sky-600 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transition duration-300 ease-in-out transform hover:-translate-y-1">
        Adicionar Nova Obra
      </button>
      {projects.length === 0 ? (
        <p className="text-center text-gray-500 text-lg py-8">Nenhuma obra cadastrada.</p>
      ) : (
        <div className="overflow-x-auto rounded-lg shadow-md border border-gray-200">
          <table className="min-w-full divide-y divide-gray-200">
            <thead className="bg-gradient-to-r from-blue-500 to-indigo-600 text-white">
              <tr>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider rounded-tl-lg">Nome da Obra</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider">Endereço</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider">Empresa Contratante</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider">Responsável</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider">Telefone</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider">Status</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider">Início Previsto</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider rounded-tr-lg">Conclusão Prevista</th>
              </tr>
            </thead>
            <tbody className="bg-white divide-y divide-gray-200">
              {projects.map((proj, index) => (
                <tr key={proj.id} className={index % 2 === 0 ? 'bg-white' : 'bg-gray-50'}>
                  <td className="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">{proj.name}</td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{proj.address || 'N/A'}</td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{proj.clientCompany || 'N/A'}</td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{proj.contactPerson || 'N/A'}</td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{proj.contactPhone || 'N/A'}</td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{proj.status || 'N/A'}</td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{proj.startDate || 'N/A'}</td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{proj.endDate || 'N/A'}</td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      )}
    </div>
  );

  const ContractorsList = () => (
    <div className="p-6 bg-white rounded-xl shadow-lg animate-fade-in">
      <h2 className="text-3xl font-extrabold mb-6 text-gray-900 text-center">Empresas Executoras Cadastradas</h2>
      <button onClick={handleAddContractor}
        className="mb-6 inline-flex items-center px-6 py-3 border border-transparent text-base font-medium rounded-full shadow-lg text-white bg-gradient-to-r from-blue-500 to-sky-500 hover:from-blue-600 hover:to-sky-600 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transition duration-300 ease-in-out transform hover:-translate-y-1">
        Adicionar Nova Empresa
      </button>
      {contractors.length === 0 ? (
        <p className="text-center text-gray-500 text-lg py-8">Nenhuma empresa executora cadastrada.</p>
      ) : (
        <div className="overflow-x-auto rounded-lg shadow-md border border-gray-200">
          <table className="min-w-full divide-y divide-gray-200">
            <thead className="bg-gradient-to-r from-blue-500 to-indigo-600 text-white">
              <tr>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider rounded-tl-lg">Nome da Empresa</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider">CNPJ</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider">Contato Principal</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider">Telefone</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider rounded-tr-lg">E-mail</th>
              </tr>
            </thead>
            <tbody className="bg-white divide-y divide-gray-200">
              {contractors.map((cont, index) => (
                <tr key={cont.id} className={index % 2 === 0 ? 'bg-white' : 'bg-gray-50'}>
                  <td className="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">{cont.name}</td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{cont.cnpj || 'N/A'}</td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{cont.mainContact || 'N/A'}</td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{cont.phone || 'N/A'}</td>
                  <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{cont.email || 'N/A'}</td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      )}
    </div>
  );

  // --- Main Render ---
  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 font-sans antialiased p-4 sm:p-6">
      <script src="https://cdn.tailwindcss.com"></script>
      <style>
        {`
          @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap');
          body {
            font-family: 'Inter', sans-serif;
          }
          /* Custom scrollbar for better aesthetics */
          ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
          }
          ::-webkit-scrollbar-track {
            background: #f1f1f1;
            border-radius: 10px;
          }
          ::-webkit-scrollbar-thumb {
            background: #888;
            border-radius: 10px;
          }
          ::-webkit-scrollbar-thumb:hover {
            background: #555;
          }

          /* Fade-in animation for tab content */
          @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
          }
          .animate-fade-in {
            animation: fadeIn 0.5s ease-out forwards;
          }
        `}
      </style>

      <header className="mb-8 bg-white shadow-xl rounded-2xl p-6 text-center">
        <h1 className="text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-indigo-600 to-purple-700 tracking-tight">
          Sistema de Controle de Visitas Técnicas
        </h1>
        <p className="mt-2 text-lg text-gray-600">Gestão eficiente de obras e segurança do trabalho</p>
      </header>

      <nav className="mb-8 flex flex-wrap justify-center gap-3 sm:gap-4 p-3 bg-white rounded-xl shadow-md">
        <button
          onClick={() => setActiveTab('visitLog')}
          className={`py-2 px-4 sm:py-3 sm:px-6 rounded-full font-semibold text-sm sm:text-base transition-all duration-300 ease-in-out transform hover:-translate-y-1 hover:shadow-lg
            ${activeTab === 'visitLog' ? 'bg-gradient-to-r from-indigo-600 to-purple-600 text-white shadow-lg' : 'bg-gray-100 text-gray-700 hover:bg-indigo-100 hover:text-indigo-800'}`}>
          Registrar Visita
        </button>
        <button
          onClick={() => setActiveTab('history')}
          className={`py-2 px-4 sm:py-3 sm:px-6 rounded-full font-semibold text-sm sm:text-base transition-all duration-300 ease-in-out transform hover:-translate-y-1 hover:shadow-lg
            ${activeTab === 'history' ? 'bg-gradient-to-r from-indigo-600 to-purple-600 text-white shadow-lg' : 'bg-gray-100 text-gray-700 hover:bg-indigo-100 hover:text-indigo-800'}`}>
          Histórico de Visitas
        </button>
        <button
          onClick={() => setActiveTab('projects')}
          className={`py-2 px-4 sm:py-3 sm:px-6 rounded-full font-semibold text-sm sm:text-base transition-all duration-300 ease-in-out transform hover:-translate-y-1 hover:shadow-lg
            ${activeTab === 'projects' ? 'bg-gradient-to-r from-indigo-600 to-purple-600 text-white shadow-lg' : 'bg-gray-100 text-gray-700 hover:bg-indigo-100 hover:text-indigo-800'}`}>
          Obras Cadastradas
        </button>
        <button
          onClick={() => setActiveTab('contractors')}
          className={`py-2 px-4 sm:py-3 sm:px-6 rounded-full font-semibold text-sm sm:text-base transition-all duration-300 ease-in-out transform hover:-translate-y-1 hover:shadow-lg
            ${activeTab === 'contractors' ? 'bg-gradient-to-r from-indigo-600 to-purple-600 text-white shadow-lg' : 'bg-gray-100 text-gray-700 hover:bg-indigo-100 hover:text-indigo-800'}`}>
          Empresas Executoras
        </button>
      </nav>

      <main>
        {activeTab === 'visitLog' && <VisitLogForm />}
        {activeTab === 'history' && <VisitHistory />}
        {activeTab === 'projects' && <ProjectsList />}
        {activeTab === 'contractors' && <ContractorsList />}
      </main>

      <footer className="mt-12 text-center text-gray-600 text-sm">
        <p>Desenvolvido para controle de visitas técnicas. Dados armazenados localmente no navegador.</p>
        <p>Para persistência de dados em nuvem, seria necessário integrar com um banco de dados (ex: Firestore).</p>
      </footer>

      <CustomAlert />
    </div>
  );
}

export default App;
